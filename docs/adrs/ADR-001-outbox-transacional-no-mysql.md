# ADR-001: Outbox transacional no MySQL existente

**Status:** Aceito
**Data:** Reunião técnica de quinta-feira, 09:00 (ver `TRANSCRICAO.md`)
**ADRs relacionadas:** [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)

---

## Contexto e Problema

Três clientes B2B pediram para ser avisados quando o status dos seus pedidos muda. Hoje eles consultam a API de pedidos periodicamente, o que deixa a integração lenta e cara ([09:00] Marcos). Para eles, qualquer entrega abaixo de 10 segundos já é "tempo real" ([09:02] Marcos).

O ponto natural de disparo é a mudança de status do pedido. No código, ela acontece numa única transação de banco que valida a transição na máquina de estados, debita ou repõe estoque, atualiza o pedido e grava o histórico de status. Essa transação já é considerada pesada ([09:04] Bruno).

O problema é decidir como tirar a notificação de dentro desse fluxo sem perder a garantia de que toda mudança de status gera um evento. Uma notificação perdida ou uma mudança desfeita por causa de um cliente fora do ar são inaceitáveis ([09:04] Bruno, [09:40] Bruno).

## Fatores de Decisão

- A mudança de status não pode esperar nem depender da disponibilidade do sistema do cliente ([09:04] Bruno).
- "Não pode ter caso de status mudar e evento não sair" ([09:40] Bruno), e um evento de uma mudança desfeita também não pode sair ([09:06] Diego).
- O time é pequeno e não quer operar infraestrutura nova ([09:07] Diego).
- A entrega precisa ficar abaixo de 10 segundos ([09:02] Marcos).
- O evento deve refletir o estado do pedido no momento da mudança, mesmo que ele mude depois ([09:52] Larissa).

## Alternativas Consideradas

1. **Outbox transacional no MySQL existente:** o evento é gravado numa tabela de saída dentro da mesma transação da mudança de status, e um worker faz a entrega depois.
2. **Disparo síncrono dentro da transação:** a chamada HTTP ao cliente é feita no próprio fluxo de mudança de status.
3. **Fila externa (Redis Streams ou similar):** o evento é publicado num broker dedicado.

## Decisão

Alternativa escolhida: **outbox transacional no MySQL existente**. Com ela, se a transação commita o evento existe, e se ela sofre rollback o evento some junto ([09:06] Diego). Isso é feito sem subir infraestrutura nova ([09:07] Diego, [09:08] Larissa).

O evento é gravado pela transação corrente, por meio de uma função que a recebe como parâmetro ([09:41] Bruno), e uma falha ao gravá-lo desfaz a mudança de status ([09:40] Bruno). Duas regras definem o que é gravado. Só entra evento se algum webhook do customer assina o novo status ([09:34] Bruno). E o payload é gravado já montado, como um retrato do pedido naquele momento ([09:52] Larissa, [09:52] Diego). A entrega HTTP fica a cargo de um worker separado ([ADR-002](ADR-002-worker-em-processo-separado-com-polling.md)).

## Prós e Contras das Alternativas

### Outbox transacional no MySQL existente
- Pró: atomicidade entre a mudança de status e o registro do evento.
- Pró: nenhuma infraestrutura nova, com o mesmo banco e o mesmo ORM.
- Contra: acrescenta escrita (e a leitura das assinaturas) a uma transação já pesada.
- Contra: a tabela cresce sem parar, e o arquivamento ficou fora do escopo ([09:08] Diego).

### Disparo síncrono dentro da transação
- Pró: latência mínima, sem componente intermediário.
- Contra: um cliente lento trava a mudança de status de outros pedidos ([09:04] Bruno).
- Contra: com o cliente fora do ar, restaria desfazer uma mudança legítima ou perder o evento ([09:04] Bruno).
- Contra: descartada como "fora de questão" ([09:06] Diego).

### Fila externa (Redis Streams ou similar)
- Pró: mecanismo de entrega reativo, sem polling.
- Contra: exige subir e operar infraestrutura nova ([09:07] Larissa).
- Contra: *overengineering* para um time pequeno ([09:07] Diego).
- Contra: publicar fora do banco não é atômico com o commit da transação.

## Consequências

**Positivas.** Toda mudança de status confirmada tem um evento correspondente, e nenhuma é desfeita por falha de cliente externo. A outbox também funciona como registro do que foi ou não entregue. O retrato gravado na inserção garante que retentativas e reprocessamentos enviem sempre o mesmo conteúdo, o que é coerente com a deduplicação por identificador de evento ([ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)).

**Negativas.** A transação de mudança de status passa a consultar as assinaturas do customer e a gravar o evento, e a reunião não definiu um teto de latência aceitável para esse acréscimo. A tabela cresce continuamente até que o arquivamento, adiado ([09:08] Diego), seja feito. Filtrar na inserção significa que um webhook criado ou alterado depois da mudança não recebe eventos retroativos. O destino de eventos já gravados para um webhook desativado não foi tratado na reunião e fica como questão em aberto no RFC.

**Trade-off explícito:** aceitamos custo extra na transação de status e alguns segundos de latência em troca de consistência forte entre "status mudou" e "evento registrado", sem operar infraestrutura nova.

## Referências

- `src/modules/orders/order.service.ts:126` (transação de mudança de status; o evento é gravado depois do histórico, linha 159)
- `src/modules/orders/order.service.ts:24` (tipo do client transacional, já repassado a funções auxiliares)
- `src/modules/orders/order.status.ts:3` (máquina de estados que define as transições que geram evento)
- `prisma/schema.prisma:74` (modelo de pedido, fonte dos dados do evento)
- `docker-compose.yml` (MySQL 8.0 como única infraestrutura existente)
- Transcrição: [09:04] Bruno, [09:06] Diego, [09:07] Larissa, [09:07] Diego, [09:08] Larissa, [09:34] Bruno, [09:40] Bruno, [09:41] Diego, [09:41] Bruno, [09:52] Larissa, [09:52] Diego
