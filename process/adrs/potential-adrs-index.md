# Índice de Potential ADRs

> Fase 2 do fluxo de ADRs, gerada com o prompt `process/prompts/02-adr-identify-adaptado.md` a partir de `process/adrs/mapping.md` e `TRANSCRICAO.md`.

## Progresso da análise

- **Fonte analisada:** `TRANSCRICAO.md`, de [09:00] a [09:53], e os pontos de integração do código mapeados em `mapping.md`.
- **Módulo:** WEBHOOKS (a criar), com impacto em ORDERS, PLATFORM, DATA e AUTH.
- **Data:** 25-09-2026.
- **Resultado:** 45 candidatos inventariados. 6 são *must document*, 1 é *consider* e 38 foram consolidados, descartados ou encaminhados a outro documento.

## Alta prioridade (must-document/)

| Título | Categoria | Score | Arquivo | ADR gerada |
| --- | --- | --- | --- | --- |
| Outbox transacional no MySQL existente | Camada de dados | 125 | [outbox-transacional-no-mysql.md](potential-adrs/must-document/WEBHOOKS/outbox-transacional-no-mysql.md) | [ADR-001](../../docs/adrs/ADR-001-outbox-transacional-no-mysql.md) |
| Worker em processo separado com polling | Infraestrutura | 110 | [worker-em-processo-separado-com-polling.md](potential-adrs/must-document/WEBHOOKS/worker-em-processo-separado-com-polling.md) | [ADR-002](../../docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md) |
| Retry com backoff exponencial e DLQ em tabela separada | Confiabilidade | 105 | [retry-com-backoff-exponencial-e-dlq.md](potential-adrs/must-document/WEBHOOKS/retry-com-backoff-exponencial-e-dlq.md) | [ADR-003](../../docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md) |
| Autenticação HMAC-SHA256 com secret por endpoint e rotação | Segurança | 125 | [autenticacao-hmac-sha256-com-secret-por-endpoint.md](potential-adrs/must-document/WEBHOOKS/autenticacao-hmac-sha256-com-secret-por-endpoint.md) | [ADR-004](../../docs/adrs/ADR-004-autenticacao-hmac-sha256-com-secret-por-endpoint.md) |
| Entrega at-least-once com X-Event-Id | Contrato de integração | 125 | [entrega-at-least-once-com-x-event-id.md](potential-adrs/must-document/WEBHOOKS/entrega-at-least-once-com-x-event-id.md) | [ADR-005](../../docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) |
| Reuso dos padrões existentes do projeto | Framework e plataforma | 125 | [reuso-dos-padroes-do-projeto.md](potential-adrs/must-document/WEBHOOKS/reuso-dos-padroes-do-projeto.md) | [ADR-006](../../docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md) |

Os 6 cobrem as 6 decisões principais listadas no enunciado (requisito 4).

## Média prioridade (consider/)

| Título | Categoria | Score | Arquivo |
| --- | --- | --- | --- |
| Modelo de autorização do módulo de webhooks | Segurança | 95 | [modelo-de-autorizacao-do-modulo-de-webhooks.md](potential-adrs/consider/WEBHOOKS/modelo-de-autorizacao-do-modulo-de-webhooks.md) |

**Decisão do time:** não vira ADR. Os riscos vão para o FDD e o PRD: CRUD aberto a qualquer usuário autenticado, falta de vínculo entre usuário e customer, e registro público que aceita ADMIN.

## Inventário completo de candidatos

Legenda da classificação:
- **MUST:** must document.
- **CONS:** consider.
- **CONSOL:** consolidado na decisão maior (Red Flag 3 ou 5).
- **DESC:** descartado como ADR.
- **ALT:** alternativa rejeitada na reunião, que entra na seção de alternativas.
- **ESCOPO:** fora de escopo ou adiado.
- **ABERTO:** não decidido na reunião.

| # | Candidato | Origem | Classificação | Destino |
| --- | --- | --- | --- | --- |
| C01 | Webhooks apenas de saída (sem entrada) | [09:02] Marcos, [09:03] Sofia | ESCOPO | PRD: fora de escopo |
| C02 | Disparo síncrono dentro do service de pedidos | [09:03] Larissa, [09:04] Bruno, [09:06] Diego | ALT | ADR Outbox, RFC |
| C03 | Outbox transacional no MySQL | [09:06] Diego, [09:08] Larissa | MUST (125) | ADR Outbox |
| C04 | Redis Streams ou fila externa | [09:07] Larissa, [09:07] Diego | ALT | ADR Outbox, RFC |
| C05 | Índices da outbox (status, `created_at`) e seus estados | [09:08] Diego | CONSOL (RF5) | ADR Outbox, FDD |
| C06 | Arquivar eventos entregues após ~30 dias | [09:08] Diego | ESCOPO | PRD: fora de escopo |
| C07 | Polling a cada 2 s | [09:09] Diego, [09:10] Larissa | CONSOL (RF3) | ADR Worker, FDD |
| C08 | Trigger de banco para reagir às inserções | [09:09] Bruno, [09:09] Diego | ALT | ADR Worker, RFC |
| C09 | Worker em processo separado (`src/worker.ts`, `npm run worker`) | [09:11] Diego, [09:11] Larissa | MUST (110) | ADR Worker |
| C10 | Worker no mesmo processo da API | [09:11] Diego | ALT | ADR Worker |
| C11 | Instância única, ordem implícita por `order_id` | [09:12] Diego, [09:13] Larissa | CONSOL (RF5) | ADR Worker, FDD |
| C12 | Vários workers (particionamento ou lock) | [09:13] Diego | ESCOPO / ABERTO | RFC: questões em aberto |
| C13 | Backoff exponencial 1m/5m/30m/2h/12h | [09:15] Diego, [09:17] Larissa | MUST (105) | ADR Retry/DLQ |
| C14 | Retry indefinido | [09:15] Diego | ALT | ADR Retry/DLQ |
| C15 | Apenas 3 tentativas | [09:16] Bruno, [09:16] Diego | ALT | ADR Retry/DLQ, RFC |
| C16 | DLQ em tabela `webhook_dead_letter` | [09:18] Diego | CONSOL (RF5) | ADR Retry/DLQ, FDD |
| C17 | DLQ como status "failed" na outbox | [09:17] Larissa, [09:18] Diego | ALT | ADR Retry/DLQ |
| C18 | Replay manual da DLQ por endpoint admin | [09:18] Diego | CONSOL (RF5) | ADR Retry/DLQ, FDD |
| C19 | HMAC-SHA256 sobre o corpo, header `X-Signature` | [09:20] Sofia, [09:22] Sofia | MUST (125) | ADR HMAC |
| C20 | Secret única por endpoint | [09:21] Sofia | CONSOL (RF5) | ADR HMAC |
| C21 | Secret global da plataforma | [09:21] Sofia | ALT | ADR HMAC, RFC |
| C22 | Rotação de secret com carência de 24 h | [09:21] Sofia, [09:22] Sofia | CONSOL (RF5) | ADR HMAC, FDD |
| C23 | URL obrigatoriamente `https` | [09:23] Sofia ("nem é decisão arquitetural") | DESC | FDD (validação), PRD (RNF) |
| C24 | Limite de payload de 64 KB, com erro | [09:23] Sofia, [09:24] Diego, [09:24] Larissa ("é só requisito não funcional") | DESC | PRD (RNF), FDD |
| C25 | Truncar payload acima do limite | [09:23] Sofia | ALT | FDD |
| C26 | Entrega at-least-once com `X-Event-Id` | [09:24] Diego, [09:26] Larissa | MUST (125) | ADR At-least-once |
| C27 | Exactly-once | [09:25] Diego | ALT | ADR At-least-once, RFC |
| C28 | Módulo `src/modules/webhooks` no padrão do projeto | [09:27] Bruno, [09:28] Diego | CONSOL (RF5) | ADR Reuso |
| C29 | Erros `AppError` com prefixo `WEBHOOK_` | [09:28] Bruno, [09:29] Larissa | CONSOL (RF5) | ADR Reuso, FDD (matriz de erros) |
| C30 | Reuso máximo dos padrões (Pino, error middleware, Zod) | [09:29] Bruno, [09:30] Larissa | MUST (125) | ADR Reuso |
| C31 | `PrismaClient` próprio no worker | [09:29] Diego, [09:30] Bruno | CONSOL (RF5) | ADR Worker |
| C32 | CRUD de configuração (POST, PATCH, DELETE, GET por customer) | [09:31] Marcos, [09:33] Bruno | DESC (RF2, requisito funcional) | PRD (RF), FDD (contratos) |
| C33 | `customer_id` no body ou path, não do JWT | [09:31] Marcos, [09:32] Larissa | CONSOL | ADR consider Autorização, FDD |
| C34 | Filtro de eventos por status, aplicado na inserção | [09:33] Marcos, [09:34] Bruno | CONSOL (RF5) | ADR Outbox, FDD |
| C35 | Histórico de entregas (`GET /webhooks/:id/deliveries`) | [09:34] Marcos | DESC (RF2, requisito funcional) | PRD (RF), FDD |
| C36 | Replay exige ADMIN com auditoria; CRUD com qualquer role | [09:36] Sofia, [09:36] Larissa, [09:37] Sofia | CONS (95) | ADR consider Autorização |
| C37 | E-mail ao cliente em falhas seguidas | [09:37] Marcos, [09:37] Larissa | ESCOPO (próxima fase) | PRD: fora de escopo |
| C38 | Rate limiting de envio ao cliente | [09:38] Diego, [09:39] Larissa | ABERTO | RFC: questões em aberto |
| C39 | Dashboard visual para o cliente | [09:39] Marcos, [09:40] Larissa | ESCOPO | PRD: fora de escopo |
| C40 | Inserção na mesma transação de `changeStatus`, rollback se falhar | [09:40] Bruno, [09:41] Diego | CONSOL (RF5) | ADR Outbox, FDD (integração) |
| C41 | Função que recebe a transação vs. injetar repository | [09:41] Bruno, [09:41] Diego | CONSOL (RF4) | ADR Reuso, FDD (integração) |
| C42 | Timeout HTTP de 10 s | [09:42] Diego | CONSOL (RF3) | ADR Retry/DLQ, FDD |
| C43 | Formato do payload JSON (campos, sem itens) | [09:43] Diego, [09:44] Bruno | DESC (score 40: fora do Step 0) | FDD (contrato) |
| C44 | Headers `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id`, `Content-Type` | [09:44] Diego, [09:44] Sofia, [09:45] Diego | DESC (contrato) | FDD (contrato) |
| C45 | UUID na outbox e payload em snapshot na inserção | [09:51] Larissa, [09:52] Larissa, [09:52] Bruno | CONSOL (RF3 e RF5) | ADR Outbox, ADR Reuso |

Pontos da reunião que não são decisões arquiteturais e não entram no inventário:
- Estimativa de três sprints ([09:46] Larissa): vai para PRD e RFC.
- Revisão de segurança de 2 dias úteis ([09:46] Sofia): vai como dependência no PRD e critério no FDD.
- Documentação no portal do desenvolvedor ([09:26] Marcos): vai como dependência no PRD.

## Resumo

- Alta prioridade: **6**.
- Média prioridade: **1**.
- Total com arquivo: **7**.
- Decisões principais do enunciado cobertas: **6 de 6**.

## Decisões do time antes da Fase 3

1. **Consider "Modelo de autorização":** fica só no FDD e no PRD como risco, e não vira ADR.
2. **"5 tentativas":** confirmado que são 5 retentativas após o envio inicial, ou seja, no máximo 6 chamadas HTTP.
3. **Data das ADRs:** referência à reunião (`TRANSCRICAO.md`, "quinta-feira, 09:00"), já que a transcrição não informa data de calendário.
4. **Demais `[NEEDS INPUT]`:** cada um virou limitação registrada nas Consequências da ADR e questão em aberto para o RFC.

### Questões levadas ao RFC

| Origem | Questão |
| --- | --- |
| ADR-001 | Destino dos eventos já gravados quando o webhook é desativado ou removido |
| ADR-001 | Teto de latência aceitável para o acréscimo na transação de mudança de status |
| ADR-002 | Recuperação de eventos presos em processamento quando o worker cai |
| ADR-002 | Monitoramento de que o worker está vivo |
| ADR-002 | Ordem por pedido quando um evento anterior está em retentativa |
| ADR-003 | Quais respostas HTTP, além da falta de resposta, contam como falha |
| ADR-003 | Registro de auditoria do replay só em log ou também persistido |
| ADR-003, ADR-005 | Se o replay mantém o identificador original do evento |
| ADR-004 | Como assinar durante a carência de 24 h, com duas secrets válidas |
| ADR-004 | `X-Timestamp` fora da assinatura |
| ADR-005 | Mesmo identificador de evento quando há dois webhooks do mesmo customer |
