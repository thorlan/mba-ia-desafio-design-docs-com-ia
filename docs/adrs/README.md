# Architectural Decision Records

Este diretório reúne as ADRs da feature **Sistema de Webhooks de Notificação de Pedidos**, no formato MADR, nomeadas como `ADR-NNN-titulo-em-kebab-case.md`.

| ADR | Decisão | Status |
| --- | --- | --- |
| [ADR-001](ADR-001-outbox-transacional-no-mysql.md) | Outbox transacional no MySQL existente | Aceito |
| [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md) | Worker em processo separado com polling | Aceito |
| [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md) | Retry com backoff exponencial e DLQ em tabela separada | Aceito |
| [ADR-004](ADR-004-autenticacao-hmac-sha256-com-secret-por-endpoint.md) | Autenticação HMAC-SHA256 com secret por endpoint e rotação | Aceito |
| [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) | Entrega at-least-once com X-Event-Id para deduplicação | Aceito |
| [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) | Reuso dos padrões existentes do projeto | Aceito |

Todas as decisões vêm da reunião registrada em [`TRANSCRICAO.md`](../../TRANSCRICAO.md). O caminho de cada uma, da identificação até a formalização, está em [`process/adrs/`](../../process/adrs/).
