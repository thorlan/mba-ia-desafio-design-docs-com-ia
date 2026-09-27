# ADR-001: Outbox transacional no MySQL existente

**Status:** Aceito
**Data:** Reunião técnica de quinta-feira, 09:00 (ver `TRANSCRICAO.md`)
**Usada por:**
- [ADR-002: Worker em processo separado com polling](./ADR-002-worker-em-processo-separado-com-polling.md)
- [ADR-003: Retry com backoff exponencial e DLQ em tabela separada](./ADR-003-retry-com-backoff-exponencial-e-dlq.md)
- [ADR-005: Entrega at-least-once com X-Event-Id para deduplicação](./ADR-005-entrega-at-least-once-com-x-event-id.md)

**Relacionada a:** [ADR-006: Reuso dos padrões existentes do projeto](./ADR-006-reuso-dos-padroes-do-projeto.md)

## Contexto e Problema

Três clientes B2B pediram para ser avisados quando o status dos seus pedidos muda. Hoje eles consultam a API de pedidos periodicamente, o que deixa a integração lenta e cara ([09:00] Marcos). Para eles, qualquer entrega abaixo de 10 segundos já é "tempo real" ([09:02] Marcos).

O ponto natural de disparo é a mudança de status do pedido. No código, ela acontece numa única transação de banco que valida a transição na máquina de estados, debita ou repõe estoque, atualiza o pedido e grava o histórico de status (`src/modules/orders/order.service.ts:131-178`). Essa transação já é considerada pesada ([09:04] Bruno).

O problema é decidir como tirar a notificação de dentro desse fluxo sem perder a garantia de que toda mudança de status gera um evento. Uma notificação perdida ou uma mudança desfeita por causa de um cliente fora do ar são inaceitáveis ([09:04] Bruno, [09:40] Bruno).

## Fatores de Decisão

- A mudança de status não pode esperar nem depender da disponibilidade do sistema do cliente ([09:04] Bruno).
- "Não pode ter caso de status mudar e evento não sair" ([09:40] Bruno), e um evento de uma mudança desfeita também não pode sair ([09:06] Diego).
- O time é pequeno e não quer operar infraestrutura nova ([09:07] Diego).
- A entrega precisa ficar abaixo de 10 segundos ([09:02] Marcos).
- O evento deve refletir o estado do pedido no momento da mudança, mesmo que ele mude depois ([09:52] Larissa).

## Alternativas Consideradas

1. **Outbox transacional no MySQL existente:** o evento é gravado numa tabela de saída dentro da mesma transação da mudança de status, e um worker faz a entrega depois ([09:06] Diego).
2. **Disparo síncrono dentro da transação:** a chamada HTTP ao cliente é feita no próprio fluxo de mudança de status ([09:03] Larissa).
3. **Fila externa (Redis Streams ou similar)** ([09:07] Larissa).

## Decisão

Alternativa escolhida: **outbox transacional no MySQL existente**, porque se a transação commita o evento existe, e se ela sofre rollback o evento some junto ([09:06] Diego), sem subir infraestrutura nova ([09:07] Diego, [09:08] Larissa).

O evento é gravado pela própria transação de mudança de status ([09:41] Bruno), e uma falha ao gravá-lo desfaz a mudança de status ([09:40] Bruno). Duas regras definem o que é gravado. Só entra evento se algum webhook do customer assina o novo status ([09:34] Bruno). E o payload é gravado já montado, como um retrato do pedido naquele momento ([09:52] Larissa, [09:52] Diego). A entrega HTTP fica a cargo de um worker separado ([ADR-002](ADR-002-worker-em-processo-separado-com-polling.md)).

## Prós e Contras das Alternativas

### Outbox transacional no MySQL existente
- Pró: se a transação commitou, o evento foi registrado; se deu rollback, some junto ([09:06] Diego).
- Pró: resolve com o MySQL existente, sem infraestrutura nova ([09:07] Diego).
- Contra: acrescenta a gravação do evento, e a consulta dos webhooks assinantes, a uma transação já pesada ([09:04] Bruno, [09:34] Bruno).
- Contra: a tabela cresce sem parar, e o arquivamento ficou fora do escopo ([09:08] Diego).

### Disparo síncrono dentro da transação
- Contra: um cliente lento trava a mudança de status de outros pedidos ([09:04] Bruno).
- Contra: com o cliente fora do ar, restaria desfazer uma mudança legítima ou perder o evento ([09:04] Bruno).
- Contra: descartada como "fora de questão" ([09:06] Diego).

### Fila externa (Redis Streams ou similar)
- Contra: exige subir e operar infraestrutura nova ([09:07] Larissa).
- Contra: *overengineering* para um time pequeno ([09:07] Diego).
- Contra: publicar fora da transação "perde a garantia toda" ([09:41] Diego).

## Consequências

**Positivas.** Toda mudança de status confirmada tem um evento correspondente ([09:06] Diego), e nenhuma é desfeita por falha de cliente externo ([09:04] Bruno). O evento reflete o estado do pedido de quando o status mudou, mesmo que o pedido mude depois ([09:52] Larissa).

**Negativas.** A transação de mudança de status passa a consultar as assinaturas do customer e a gravar o evento ([09:34] Bruno, [09:40] Bruno), e a reunião não definiu um teto de latência aceitável para esse acréscimo. Os eventos acumulam na tabela ([09:07] Bruno), e o arquivamento ficou fora do escopo ([09:08] Diego). O destino de eventos já gravados para um webhook desativado não foi tratado na reunião e fica como questão em aberto no RFC.

**Trade-off explícito:** aceitamos mais trabalho numa transação já pesada ([09:04] Bruno) e até 2 segundos de latência ([09:10] Larissa) em troca de não ter "inconsistência possível" entre a mudança de status e o evento ([09:06] Diego), sem subir infraestrutura nova ([09:07] Diego).

## Referências

- `src/modules/orders/order.service.ts:126` (transação de mudança de status; o evento é gravado depois do histórico, linha 159)
- `src/modules/orders/order.service.ts:24` (tipo do client transacional, já repassado a funções auxiliares)
- `src/modules/orders/order.status.ts:3` (máquina de estados que define as transições que geram evento)
- `prisma/schema.prisma:74` (modelo de pedido, fonte dos dados do evento)
- `docker-compose.yml:3` (MySQL 8.0 como única infraestrutura existente)
- Transcrição: [09:00] Marcos, [09:02] Marcos, [09:03] Larissa, [09:04] Bruno, [09:06] Diego, [09:07] Diego, [09:07] Larissa, [09:07] Bruno, [09:08] Larissa, [09:08] Diego, [09:10] Larissa, [09:34] Bruno, [09:40] Bruno, [09:41] Bruno, [09:41] Diego, [09:52] Larissa, [09:52] Diego
