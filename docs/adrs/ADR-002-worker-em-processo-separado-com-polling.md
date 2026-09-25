# ADR-002: Worker em processo separado com polling

**Status:** Aceito
**Data:** Reunião técnica de quinta-feira, 09:00 (ver `TRANSCRICAO.md`)
**ADRs relacionadas:** [ADR-001](ADR-001-outbox-transacional-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)

---

## Contexto e Problema

Com os eventos gravados na outbox ([ADR-001](ADR-001-outbox-transacional-no-mysql.md)), algum componente precisa ler os pendentes e fazer as chamadas HTTP aos clientes. A meta de negócio é entregar em menos de 10 segundos ([09:02] Marcos).

O MySQL não tem um mecanismo nativo para avisar processos externos de que uma linha foi inserida, como o NOTIFY/LISTEN do Postgres. Uma trigger só executa SQL e não acorda outro processo ([09:09] Diego).

Hoje a aplicação roda num único processo HTTP, com desligamento gracioso. Se a entrega de webhooks rodar dentro dele, cada deploy ou restart da API interrompe as entregas ([09:11] Diego).

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

Alternativa escolhida: **worker em processo separado, com polling a cada 2 segundos**. Os 2 segundos cabem com folga na meta de 10 ([09:09] Diego, [09:10] Marcos), e a decisão foi registrada em [09:10] Larissa.

O worker tem entry point próprio e um script de execução ao lado do servidor HTTP ([09:11] Larissa). Usa o mesmo banco e o mesmo código, mas com a sua própria conexão de ORM, porque essa conexão é por processo ([09:30] Bruno). A cada ciclo ele lê em lote pequeno os eventos pendentes mais antigos, processa e registra o resultado ([09:08] Diego, [09:09] Diego).

Roda uma única instância. A ordem de entrega segue a ordem de gravação, o que dá ordem por pedido, sem garantia de ordem global ([09:12] Diego, [09:13] Larissa). Escalar para vários workers ficou para o futuro ([09:13] Diego).

## Prós e Contras das Alternativas

### Worker em processo separado, com polling
- Pró: entregas isoladas de deploys e restarts da API.
- Pró: só tecnologias já usadas no projeto.
- Contra: consulta o banco a cada 2 segundos, mesmo sem eventos.
- Contra: a instância única é ponto único de falha e limita a vazão.

### Trigger no banco
- Pró: reagiria no momento da inserção.
- Contra: a trigger do MySQL não notifica processo externo ([09:09] Diego).
- Contra: exigiria um desvio improvisado, como escrever em arquivo ou chamar um endpoint, que "fica esquisito" ([09:09] Diego).

### Worker dentro do processo da API
- Pró: um único processo para implantar e monitorar.
- Contra: um restart da API derruba o worker junto ([09:11] Diego).
- Contra: disputa recursos com o atendimento HTTP.

## Consequências

**Positivas.** O pior caso de espera pela leitura fica em torno de 2 segundos, dentro da meta. Deploys da API e do worker são independentes. E, com o worker parado, os eventos acumulam na outbox sem se perder.

**Negativas.** O sistema passa a ter dois processos para implantar e operar, e o worker exige a mesma configuração de ambiente da API. A ordem por pedido vale apenas no caminho feliz: quando um evento entra em retentativa ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)), o seguinte do mesmo pedido pode ser entregue antes. Com uma instância e timeout de 10 segundos por chamada ([09:42] Diego), um cliente lento atrasa os demais do mesmo ciclo. A reunião não tratou a recuperação de eventos presos em processamento quando o worker cai, nem o monitoramento de que o worker está vivo. Os dois pontos ficam como questões em aberto no RFC.

**Trade-off explícito:** trocamos reatividade imediata e escala horizontal por simplicidade operacional. A latência de até ~2 segundos é aceita ([09:10] Larissa), e a garantia de ordem é por pedido e só no caminho feliz.

## Referências

- `src/server.ts:6` (entry point atual, com desligamento gracioso nas linhas 20-21; modelo para o novo entry point)
- `src/config/database.ts:4` (criação do client do ORM; cada processo cria o seu)
- `src/config/env.ts:27` (validação do ambiente na carga, herdada pelo worker)
- `package.json:10` (scripts de execução, onde entra o script do worker)
- Transcrição: [09:02] Marcos, [09:08] Diego, [09:09] Diego, [09:10] Larissa, [09:10] Marcos, [09:11] Diego, [09:11] Larissa, [09:12] Diego, [09:13] Diego, [09:13] Larissa, [09:14] Marcos, [09:30] Bruno, [09:42] Diego
