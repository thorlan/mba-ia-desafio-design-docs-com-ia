# Índice das ADRs

As ADRs da feature **Sistema de Webhooks de Notificação de Pedidos** ficam em [`docs/adrs/`](../../docs/adrs/), no formato MADR, nomeadas como `ADR-NNN-titulo-em-kebab-case.md`. Este índice fica aqui, e não em `docs/adrs/`, para que aquela pasta contenha só os arquivos ADR, como pede o enunciado.

| ADR | Decisão | Status |
| --- | --- | --- |
| [ADR-001](../../docs/adrs/ADR-001-outbox-transacional-no-mysql.md) | Outbox transacional no MySQL existente | Aceito |
| [ADR-002](../../docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md) | Worker em processo separado com polling | Aceito |
| [ADR-003](../../docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md) | Retry com backoff exponencial e DLQ em tabela separada | Aceito |
| [ADR-004](../../docs/adrs/ADR-004-autenticacao-hmac-sha256-com-secret-por-endpoint.md) | Autenticação HMAC-SHA256 com secret por endpoint e rotação | Aceito |
| [ADR-005](../../docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) | Entrega at-least-once com X-Event-Id para deduplicação | Aceito |
| [ADR-006](../../docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md) | Reuso dos padrões existentes do projeto | Aceito |

Todas as decisões vêm da reunião registrada em [`TRANSCRICAO.md`](../../TRANSCRICAO.md). O caminho de cada uma segue as quatro fases do plugin `adrs-management`:

1. **Map:** [`mapping.md`](mapping.md).
2. **Identify:** [`potential-adrs-index.md`](potential-adrs-index.md) e os dossiês em [`potential-adrs/`](potential-adrs/). Os dossiês já formalizados ficam em `potential-adrs/done/`.
3. **Generate:** as ADRs em `docs/adrs/`.
4. **Link:** relações tipadas nos cabeçalhos, com relatório em [`reports/`](reports/).
