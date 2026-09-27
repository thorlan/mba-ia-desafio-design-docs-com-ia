# ADR-002: Worker em processo separado com polling

**Status:** Aceito
**Data:** Reunião técnica de quinta-feira, 09:00 (ver `TRANSCRICAO.md`)
**Depende de:** [ADR-001: Outbox transacional no MySQL existente](./ADR-001-outbox-transacional-no-mysql.md)
**Usada por:**
- [ADR-003: Retry com backoff exponencial e DLQ em tabela separada](./ADR-003-retry-com-backoff-exponencial-e-dlq.md)
- [ADR-005: Entrega at-least-once com X-Event-Id para deduplicação](./ADR-005-entrega-at-least-once-com-x-event-id.md)

**Relacionada a:**
- [ADR-004: Autenticação HMAC-SHA256 com secret por endpoint e rotação](./ADR-004-autenticacao-hmac-sha256-com-secret-por-endpoint.md)
- [ADR-006: Reuso dos padrões existentes do projeto](./ADR-006-reuso-dos-padroes-do-projeto.md)

## Contexto e Problema

Com os eventos gravados na outbox ([ADR-001](ADR-001-outbox-transacional-no-mysql.md)), algum componente precisa ler os pendentes e fazer as chamadas HTTP aos clientes. A meta de negócio é entregar em menos de 10 segundos ([09:02] Marcos).

O MySQL não tem um mecanismo nativo para avisar processos externos de que uma linha foi inserida, como o NOTIFY/LISTEN do Postgres. Uma trigger só executa SQL e não acorda outro processo ([09:09] Diego).

Hoje a aplicação roda num único processo HTTP, com desligamento gracioso (`src/server.ts:6-21`). Se a entrega de webhooks rodar dentro dele, cada deploy ou restart da API interrompe as entregas ([09:11] Diego).

## Fatores de Decisão

- A entrega precisa ficar abaixo de 10 segundos ([09:02] Marcos).
- O banco disponível não notifica processos externos ([09:09] Diego).
- O ciclo de vida das entregas não pode depender do ciclo de vida da API ([09:11] Diego).
- Mesmo banco e mesma stack, sem tecnologia nova ([09:11] Diego).
- Os clientes querem saber de cada pedido e nunca pediram ordem global dos eventos ([09:14] Marcos).

## Alternativas Consideradas

1. **Worker em processo separado, com polling a cada 2 segundos.**
2. **Trigger no banco para reagir às inserções.**
3. **Worker dentro do processo da API.**

## Decisão

Alternativa escolhida: **worker em processo separado, com polling a cada 2 segundos**, porque isola as entregas do ciclo de vida da API ([09:11] Diego) e os 2 segundos cabem com folga na meta de 10 ([09:09] Diego, [09:10] Marcos). A decisão foi registrada em [09:10] Larissa.

O worker roda como processo próprio ao lado do servidor HTTP ([09:11] Larissa), com o mesmo banco e a mesma stack ([09:11] Diego). A cada ciclo ele lê em lote pequeno os eventos pendentes mais antigos, processa e registra o resultado ([09:08] Diego, [09:09] Diego). Roda uma única instância. A ordem de entrega segue a ordem de gravação, o que dá ordem por pedido, sem garantia de ordem global ([09:12] Diego, [09:13] Larissa). Escalar para vários workers ficou para o futuro ([09:13] Diego).

## Prós e Contras das Alternativas

### Worker em processo separado, com polling
- Pró: um restart da API não derruba o worker ([09:11] Diego).
- Pró: mesmo banco e mesma stack ([09:11] Diego).
- Contra: com vários workers em paralelo, perde a garantia de ordem ([09:12] Diego).

### Trigger no banco
- Pró: seria "mais reativo" ([09:09] Bruno).
- Contra: a trigger do MySQL não notifica processo externo ([09:09] Diego).
- Contra: exigiria um desvio improvisado, como escrever em arquivo ou chamar um endpoint, que "fica esquisito" ([09:09] Diego).

### Worker dentro do processo da API
- Contra: um restart da API derruba o worker junto ([09:11] Diego).
- Contra: contraria o pedido explícito de processo separado: "Só não pode ser o mesmo processo" ([09:11] Diego).

## Consequências

**Positivas.** A latência mínima fica em 2 segundos no pior caso ([09:10] Larissa), dentro da meta de 10 ([09:09] Diego). Um restart da API não derruba o worker ([09:11] Diego).

**Negativas.** O sistema passa a ter dois processos ([09:11] Diego), e o worker usa a mesma configuração de banco da API ([09:30] Bruno, `src/config/env.ts:3-10`). A reunião aceitou ordem só por pedido e enquanto houver um único worker ([09:13] Larissa), mas não tratou a ordem quando um evento está em retentativa ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)). A reunião também não tratou a recuperação de eventos presos em processamento quando o worker cai, nem o monitoramento de que o worker está vivo. Os três pontos ficam como questões em aberto no RFC.

**Trade-off explícito:** aceitamos 2 segundos de latência no pior caso ([09:10] Larissa) e uma instância só, com ordem por pedido enquanto for single-worker ([09:13] Larissa), em troca de usar a mesma stack ([09:11] Diego) e de não perder o worker quando a API reinicia ([09:11] Diego).

## Referências

- `src/server.ts:6` (entry point atual, com desligamento gracioso nas linhas 20-21; modelo para o novo entry point)
- `src/config/database.ts:4` (criação do client do ORM; cada processo cria o seu)
- `src/config/env.ts:27` (validação do ambiente na carga, herdada pelo worker)
- `package.json:10` (scripts de execução, onde entra o script do worker)
- Transcrição: [09:02] Marcos, [09:08] Diego, [09:09] Diego, [09:09] Bruno, [09:10] Marcos, [09:10] Larissa, [09:11] Diego, [09:11] Larissa, [09:12] Diego, [09:13] Larissa, [09:13] Diego, [09:14] Marcos, [09:30] Bruno
