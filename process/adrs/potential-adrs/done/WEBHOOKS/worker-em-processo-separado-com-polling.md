# Potential ADR: Worker em processo separado com polling

**Módulo:** WEBHOOKS (toca PLATFORM)
**Categoria:** Infraestrutura / Runtime
**Prioridade:** Must Document (Score: 110)
**Data de identificação:** 25-09-2026

---

## O que foi identificado

A leitura da outbox e o envio HTTP ficam num **worker que roda como processo Node separado da API**. Ele tem entry point próprio (`src/worker.ts`, a criar), é executado por `npm run worker`, usa o mesmo banco e a mesma stack, mas com uma instância própria de `PrismaClient`, porque o client é por processo.

O worker funciona por **polling a cada 2 segundos**: busca em lote pequeno os eventos pendentes mais antigos, processa e marca o resultado. A escolha vem de uma limitação do MySQL, que não tem mecanismo nativo de notificação a processos externos. O polling de 2 s atende com folga o requisito de negócio de menos de 10 s.

Roda **um único worker**. A ordem de entrega segue o `created_at` da outbox, o que dá ordem implícita por `order_id`, sem garantia de ordem global. Escalar para vários workers ficou para o futuro. Os parâmetros (2 s, lote pequeno, instância única) foram consolidados aqui por Red Flag 3 e 5.

## Por que merece uma ADR

- **Impacto:** cria uma segunda unidade de execução e implantação. O sistema deixa de ser um processo só.
- **Trade-offs:** isolamento de ciclo de vida e simplicidade, em troca de consultas constantes ao banco, ponto único de falha e ordem garantida só no caminho feliz.
- **Complexidade:** baixa. É o mesmo código, o mesmo banco e as mesmas bibliotecas.
- **Conhecimento do time:** deploy, operação e diagnóstico precisam saber que existe um segundo processo e que as entregas param se ele parar.
- **Implicações futuras:** escalar horizontalmente exige particionar por `order_id` ou usar lock, o que exige nova decisão.

### Pontuação
| Dimensão | Nota | Justificativa |
| --- | --- | --- |
| Base (Step 0 adaptado) | 75 | Fechada como decisão ([09:10] Larissa: "Vamos registrar isso como uma decisão"), categoria infraestrutura (novo processo) |
| Escopo e impacto | 10 | PLATFORM (entry point, scripts) e WEBHOOKS |
| Custo de mudança | 10 | Trocar o mecanismo de disparo ou embutir na API: 1 a 2 semanas |
| Conhecimento do time | 15 | Importante para operação e para quem mexer no módulo |
| **Total** | **110** | Regra dos 3 E's atendida |

## Evidências

### Na transcrição
- [09:09] Diego: "Polling em loop. A cada 2 segundos, busca os eventos pendentes mais antigos, processa, marca."
- [09:09] Diego: "MySQL não tem listener nativo tipo o NOTIFY/LISTEN do Postgres. Trigger no banco [...] não notifica processo externo"
- [09:10] Larissa: "Worker em polling, 2s. [...] Aceitamos."
- [09:11] Diego: "o worker tem que rodar como processo separado [...] Senão se a API reinicia, perde o worker."
- [09:11] Larissa: "criar um src/worker.ts e um script 'npm run worker'"
- [09:11] Diego: "Sim, mesmo banco, mesma stack. Só não pode ser o mesmo processo."
- [09:12] Diego: "Se a gente escala pra múltiplos workers em paralelo no futuro, perde a garantia. Por enquanto, single-worker e ordering implícita por order_id."
- [09:13] Larissa: "Não é garantia de ordering global, só por order_id e enquanto for single-worker."
- [09:28] Bruno: lógica em `src/modules/webhooks/webhook.worker.ts` ou `webhook.processor.ts`
- [09:30] Bruno: "PrismaClient é por processo. Mesmo banco, mesma DATABASE_URL, mas instância nova"

### No código
- [`src/server.ts`](../../../../../src/server.ts), linhas 6-27: único entry point hoje, com shutdown gracioso (`SIGINT`, `SIGTERM`, `prisma.$disconnect()`). É o modelo para o worker.
- [`src/config/database.ts`](../../../../../src/config/database.ts), linhas 4-10: `createPrismaClient` e o singleton `prisma`. Cada processo que importa o módulo ganha sua instância.
- [`src/config/env.ts`](../../../../../src/config/env.ts): valida todas as variáveis na carga (inclusive `JWT_SECRET`) e encerra o processo se faltar alguma. O worker herda essa exigência.
- [`package.json`](../../../../../package.json), linhas 10-21: scripts `dev` e `start` apontam só para `server`. O script `worker` é novo.

### Linha do tempo na reunião
- **[09:08] a [09:10]:** a pergunta "como o worker lê isso?", o polling de 2 s, o descarte da trigger e o registro da decisão.
- **[09:11]:** o processo separado e o entry point.
- **[09:12] a [09:14]:** a ordem por pedido, instância única, e a escala adiada.
- **[09:28] a [09:30]:** a localização do código e o `PrismaClient` próprio.
- **[09:48]:** o resumo final.

### Alternativas observadas
1. **Trigger no banco para reagir às inserções:** proposta em [09:09] Bruno, descartada em [09:09] Diego.
2. **Worker dentro do processo da API:** descartado em [09:11] Diego.
3. **Vários workers em paralelo (particionamento por `order_id` ou lock pessimista):** adiado em [09:13] Diego ("problema do futuro").

## Questões para a ADR

- A retentativa agendada de um evento (1 min a 12 h) deixa o evento seguinte do mesmo pedido sair antes, o que quebra a "ordem por order_id" mesmo com um só worker. A ordem deve bloquear eventos posteriores do mesmo pedido enquanto houver um anterior em retry, ou a limitação é aceita? [NEEDS INPUT: Diego, Larissa]
- Com um só worker e timeout de 10 s ([09:42] Diego), um cliente lento atrasa os demais. O processamento do lote é sequencial ou concorrente?
- Eventos que ficam em "processando" quando o worker cai no meio do envio: como são recuperados? A reunião não tratou.
- Como detectar que o worker parou? A reunião não definiu monitoramento de liveness.

## Potential ADRs relacionados
- [Outbox transacional no MySQL](outbox-transacional-no-mysql.md): fonte dos eventos.
- [Retry com backoff exponencial e DLQ](retry-com-backoff-exponencial-e-dlq.md): o worker executa a política de retry.

## Notas adicionais
A frase "A latência mínima vai ser 2 segundos no pior caso" ([09:10] Larissa) é contraditória ao pé da letra. O sentido, confirmado por [09:09] Diego, é que o pior caso de espera pela leitura é de cerca de 2 s.
