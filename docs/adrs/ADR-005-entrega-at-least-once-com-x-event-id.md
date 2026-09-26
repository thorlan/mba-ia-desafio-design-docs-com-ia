# ADR-005: Entrega at-least-once com X-Event-Id para deduplicação

**Status:** Aceito
**Data:** Reunião técnica de quinta-feira, 09:00 (ver `TRANSCRICAO.md`)
**Depende de:**
- [ADR-001: Outbox transacional no MySQL existente](./ADR-001-outbox-transacional-no-mysql.md)
- [ADR-002: Worker em processo separado com polling](./ADR-002-worker-em-processo-separado-com-polling.md)
- [ADR-003: Retry com backoff exponencial e DLQ em tabela separada](./ADR-003-retry-com-backoff-exponencial-e-dlq.md)

**Relacionada a:** [ADR-004: Autenticação HMAC-SHA256 com secret por endpoint e rotação](./ADR-004-autenticacao-hmac-sha256-com-secret-por-endpoint.md)

## Contexto e Problema

A combinação de outbox ([ADR-001](ADR-001-outbox-transacional-no-mysql.md)), worker ([ADR-002](ADR-002-worker-em-processo-separado-com-polling.md)) e retentativas ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)) torna possível que o mesmo evento chegue mais de uma vez ao cliente ([09:24] Diego). Isso acontece, por exemplo, quando o cliente processa a requisição mas responde depois do limite de 10 segundos ([09:42] Diego), ou quando o worker para entre o envio e o registro do sucesso.

A plataforma precisa declarar qual garantia de entrega oferece e dar ao cliente um meio de reconhecer repetições. Sem isso, cada integração trataria duplicatas de um jeito, ou nem trataria.

## Fatores de Decisão

- Nunca perder uma mudança de status, mesmo que isso gere repetições ([09:24] Diego).
- Evitar coordenação entre plataforma e cliente para confirmar entregas ([09:25] Diego).
- Seguir o padrão que os clientes já conhecem de outros provedores ([09:25] Diego).
- Dar ao cliente um identificador estável para reconhecer repetições ([09:25] Diego).

## Alternativas Consideradas

1. **At-least-once, com identificador único de evento no header `X-Event-Id`.**
2. **Exactly-once.**

## Decisão

Alternativa escolhida: **entrega at-least-once, com deduplicação pelo cliente usando o `X-Event-Id`** ([09:26] Larissa), porque nunca perde um evento ([09:24] Diego) e segue o padrão de mercado sem exigir coordenação com o cliente ([09:25] Diego).

Cada evento recebe um UUID no momento em que é gravado na outbox. Ele é único por evento ([09:25] Diego) e é enviado no header `X-Event-Id` e também dentro do payload ([09:43] Diego). O cliente deve estar preparado para receber o mesmo evento mais de uma vez e descartar as repetições por esse identificador ([09:24] Diego). O comportamento será documentado em destaque no portal do desenvolvedor ([09:26] Marcos).

## Prós e Contras das Alternativas

### At-least-once com `X-Event-Id`
- Pró: simples e coerente com outbox e retentativas: na dúvida, reenvia.
- Pró: é o padrão de mercado ("Stripe faz assim, GitHub faz assim", [09:25] Diego).
- Contra: transfere ao cliente a responsabilidade de deduplicar ([09:25] Sofia).

### Exactly-once
- Pró: o cliente nunca vê repetições.
- Contra: exige coordenação dos dois lados ([09:25] Diego).
- Contra: é bem mais complexo de construir e operar.
- Contra: resolve um problema que at-least-once com identificador já cobre em "99% dos casos" ([09:25] Diego).

## Consequências

**Positivas.** Nenhum evento é perdido por falha de confirmação. A plataforma continua simples, e o contrato com os clientes segue um modelo conhecido. Como o payload é gravado já montado ([ADR-001](ADR-001-outbox-transacional-no-mysql.md)), todo reenvio do mesmo evento carrega o mesmo conteúdo e o mesmo identificador.

**Negativas.** O cliente precisa guardar os identificadores já processados, e um cliente que não deduplique pode processar a mesma mudança duas vezes. Isso torna a documentação no portal parte essencial da entrega ([09:26] Marcos). A reunião não definiu se o reprocessamento manual da DLQ ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)) mantém o identificador original, nem se um evento enviado a dois webhooks do mesmo customer usa o mesmo identificador nos dois. Os dois pontos ficam como questões em aberto no RFC.

**Trade-off explícito:** aceitamos repetições eventuais, e o custo de deduplicação do lado do cliente, em troca de nunca perder um evento e de manter a plataforma simples.

## Referências

- `src/middlewares/request-logger.middleware.ts:2` (geração de UUID já usada no projeto)
- `prisma/schema.prisma:26` (padrão de identificadores UUID das entidades)
- `src/modules/orders/order.service.ts:131` (transação em que o evento e seu identificador são gravados)
- Transcrição: [09:24] Diego, [09:25] Diego, [09:25] Sofia, [09:26] Marcos, [09:26] Larissa, [09:42] Diego, [09:43] Diego
