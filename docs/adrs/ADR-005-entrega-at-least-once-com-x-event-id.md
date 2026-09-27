# ADR-005: Entrega at-least-once com X-Event-Id para deduplicação

**Status:** Aceito
**Data:** Reunião técnica de quinta-feira, 09:00 (ver `TRANSCRICAO.md`)
**Depende de:**
- [ADR-001: Outbox transacional no MySQL existente](./ADR-001-outbox-transacional-no-mysql.md)
- [ADR-002: Worker em processo separado com polling](./ADR-002-worker-em-processo-separado-com-polling.md)
- [ADR-003: Retry com backoff exponencial e DLQ em tabela separada](./ADR-003-retry-com-backoff-exponencial-e-dlq.md)

**Relacionada a:** [ADR-004: Autenticação HMAC-SHA256 com secret por endpoint e rotação](./ADR-004-autenticacao-hmac-sha256-com-secret-por-endpoint.md)

## Contexto e Problema

A combinação de outbox ([ADR-001](ADR-001-outbox-transacional-no-mysql.md)), worker ([ADR-002](ADR-002-worker-em-processo-separado-com-polling.md)) e retentativas ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)) torna possível que o mesmo evento chegue mais de uma vez ao cliente: "Pode acontecer de o cliente receber o mesmo evento duas vezes" ([09:24] Diego).

A plataforma precisa declarar qual garantia de entrega oferece e dar ao cliente um meio de reconhecer repetições ([09:24] Diego, [09:25] Bruno).

## Fatores de Decisão

- Garantir at-least-once, mesmo que o cliente receba o mesmo evento duas vezes ([09:24] Diego).
- Evitar coordenação entre plataforma e cliente para confirmar entregas ([09:25] Diego).
- Seguir o padrão que os clientes já conhecem de outros provedores ([09:25] Diego).
- Dar ao cliente um identificador estável para reconhecer repetições ([09:25] Diego).

## Alternativas Consideradas

1. **At-least-once, com identificador único de evento no header `X-Event-Id`.**
2. **Exactly-once.**

## Decisão

Alternativa escolhida: **entrega at-least-once, com deduplicação pelo cliente usando o `X-Event-Id`** ([09:26] Larissa), porque garante at-least-once ([09:24] Diego) e segue o padrão de mercado sem exigir coordenação com o cliente ([09:25] Diego).

Cada evento recebe um UUID no momento em que é gravado na outbox ([09:25] Diego). Ele é único por evento ([09:25] Diego) e é enviado no header `X-Event-Id` e também dentro do payload ([09:43] Diego). O cliente deve estar preparado para receber o mesmo evento mais de uma vez e descartar as repetições por esse identificador ([09:24] Diego). O comportamento será documentado em destaque no portal do desenvolvedor ([09:26] Marcos).

## Prós e Contras das Alternativas

### At-least-once com `X-Event-Id`
- Pró: é o padrão de mercado ("Stripe faz assim, GitHub faz assim", [09:25] Diego).
- Contra: transfere ao cliente a responsabilidade de deduplicar ([09:25] Sofia).

### Exactly-once
- Contra: "exigiria coordenação dos dois lados e fica muito mais complexo" ([09:25] Diego).
- Contra: resolve um problema que at-least-once com identificador já cobre em "99% dos casos" ([09:25] Diego).

## Consequências

**Positivas.** O contrato com os clientes segue o padrão de mercado ([09:25] Diego) e evita a coordenação dos dois lados que exactly-once exigiria ([09:25] Diego).

**Negativas.** O cliente precisa estar preparado para receber o mesmo evento duas vezes ([09:24] Diego) e deduplicar do lado dele ([09:25] Diego): "Isso joga responsabilidade pro cliente" ([09:25] Sofia). O comportamento será documentado em destaque no portal ([09:26] Marcos). A reunião não definiu se o reprocessamento manual da DLQ ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)) mantém o identificador original, nem se um evento enviado a dois webhooks do mesmo customer usa o mesmo identificador nos dois. Os dois pontos ficam como questões em aberto no RFC.

**Trade-off explícito:** aceitamos repetições e a deduplicação do lado do cliente ([09:24] Diego, [09:25] Sofia) em troca de não precisar de exactly-once, que "fica muito mais complexo" ([09:25] Diego).

## Referências

- `src/middlewares/request-logger.middleware.ts:2` (geração de UUID já usada no projeto)
- `prisma/schema.prisma:26` (padrão de identificadores UUID das entidades)
- `src/modules/orders/order.service.ts:131` (transação em que o evento e seu identificador são gravados)
- Transcrição: [09:24] Diego, [09:25] Bruno, [09:25] Diego, [09:25] Sofia, [09:26] Larissa, [09:26] Marcos, [09:43] Diego
