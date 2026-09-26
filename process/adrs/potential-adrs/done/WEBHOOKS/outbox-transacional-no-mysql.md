# Potential ADR: Outbox transacional no MySQL existente

**Módulo:** WEBHOOKS (toca ORDERS e DATA)
**Categoria:** Arquitetura / Camada de dados
**Prioridade:** Must Document (Score: 125)
**Data de identificação:** 25-09-2026

---

## O que foi identificado

A reunião decidiu que o evento de mudança de status não é disparado de forma síncrona. Ele é registrado numa tabela `webhook_outbox` **dentro da mesma transação SQL** que atualiza o pedido e grava o histórico, e um worker separado lê essa tabela e faz as chamadas HTTP. A garantia buscada é de consistência: se a transação commitou, o evento existe; se deu rollback, o evento some junto.

A escolha foi usar o **MySQL que a aplicação já usa**, sem subir broker ou fila externa, com o argumento de que o time é pequeno. O ponto de integração é `changeStatus`, que já concentra numa transação só a validação da transição, o estoque, o `update` em `orders` e o `insert` em `order_status_history`.

Duas decisões menores fecham o que vai para a outbox e foram consolidadas aqui (Red Flag 5, componentes da decisão maior):
- **Filtro na inserção:** só entra evento se algum webhook do customer assina aquele status.
- **Snapshot:** o payload é gravado já renderizado no momento da inserção.

## Por que merece uma ADR

- **Impacto:** passa a acoplar a mudança de status de qualquer pedido à escrita de um evento. Uma falha ao inserir o evento desfaz a mudança de status.
- **Trade-offs:** consistência forte sem infraestrutura nova, em troca de uma escrita (e uma leitura de configuração) a mais numa transação já pesada, crescimento contínuo da tabela e latência limitada pelo polling.
- **Complexidade:** baixa na infraestrutura e média no código. A atomicidade depende de a inserção usar o client da transação corrente.
- **Conhecimento do time:** quem mexer no ciclo de vida do pedido precisa saber que a transação agora também produz eventos.
- **Implicações futuras:** o arquivamento de linhas antigas ficou para depois e é dívida operacional. Trocar por um broker exigiria reimplementar a garantia de atomicidade.

### Pontuação
| Dimensão | Nota | Justificativa |
| --- | --- | --- |
| Base (Step 0 adaptado) | 75 | Fechada como decisão ([09:08] Larissa: "Tá decidido então: outbox em MySQL") e pertence às categorias camada de dados e infraestrutura |
| Escopo e impacto | 15 | ORDERS, WEBHOOKS, worker e DATA (3 a 4 módulos) |
| Custo de mudança | 15 | Migrar para broker e recriar a atomicidade: 2 a 8 semanas |
| Conhecimento do time | 20 | Crítico para toda feature que altere o ciclo de vida do pedido |
| **Total** | **125** | Regra dos 3 E's atendida: estrutural, evidente e estável |

## Evidências

### Na transcrição
- [09:04] Bruno: "Síncrono não rola. A transação de mudança de status hoje já é pesada [...] atualiza orders, insere na order_status_history, decrementa stock_quantity"
- [09:04] Bruno: "se o cliente tiver fora do ar, o que a gente faz, dá rollback na mudança de status? Não dá."
- [09:06] Diego: "dentro da mesma transação SQL que atualiza orders e order_status_history, a gente também insere uma linha numa tabela tipo webhook_outbox"
- [09:07] Diego: "Subir Redis Cluster pra isso é overengineering. Outbox no MySQL existente resolve."
- [09:08] Diego: índice no campo de status ("pendente, processando, falhou, entregue") e em `created_at`; arquivar entregues "depois de 30 dias ou assim, fora do escopo dessa feature"
- [09:08] Larissa: "Tá decidido então: outbox em MySQL."
- [09:34] Bruno: filtrar "Na inserção. Se nenhum webhook do customer quer aquele status, nem insere."
- [09:40] Bruno: "Se a outbox falhar de inserir, rollback. Não pode ter caso de status mudar e evento não sair."
- [09:41] Diego: "Se ficar fora da transação, perde a garantia toda."
- [09:41] Bruno: "publishWebhookEvent(tx, order, fromStatus, toStatus) que aceita o tx client da transação atual"
- [09:52] Larissa, Diego e Bruno: payload "renderizado já, na hora da inserção", "snapshot na inserção", "Decidido."

### No código
- [`src/modules/orders/order.service.ts`](../../../../../src/modules/orders/order.service.ts), linhas 126-179: `changeStatus` com `prisma.$transaction` (131), `tx.order.update` (158) e `tx.orderStatusHistory.create` (159). É o ponto de inserção.
- [`src/modules/orders/order.service.ts`](../../../../../src/modules/orders/order.service.ts), linha 24: `type TxClient = Prisma.TransactionClient`, padrão já usado por helpers que recebem a transação (205, 234, 245).
- [`src/modules/orders/order.status.ts`](../../../../../src/modules/orders/order.status.ts), linhas 3-37: máquina de estados que define quais transições geram eventos.
- [`prisma/schema.prisma`](../../../../../prisma/schema.prisma): novas tabelas seguem o padrão `@db.Char(36)` e `@@index`.
- [`docker-compose.yml`](../../../../../docker-compose.yml): o único serviço de infraestrutura é o MySQL 8.0.

### Linha do tempo na reunião
- **[09:03] a [09:04]:** a pergunta "síncrono ou fila/outbox" e o descarte do envio síncrono.
- **[09:06] a [09:08]:** a proposta da outbox, o descarte do Redis Streams e o fechamento da decisão.
- **[09:34]:** o filtro na inserção.
- **[09:40] a [09:41]:** a atomicidade com `changeStatus`.
- **[09:48]:** o resumo final confirma a decisão.
- **[09:51] a [09:52]:** decididos o UUID e o snapshot.

### Alternativas observadas
1. **Disparo síncrono no service de pedidos:** descartado em [09:04] Bruno e [09:06] Diego.
2. **Redis Streams ou fila externa:** descartado em [09:07] Larissa e [09:07] Diego (infra nova, *overengineering*).
3. **Registrar o evento fora da transação:** descartado em [09:41] Diego.
4. **Filtrar no envio em vez de na inserção:** descartado em [09:34] Bruno e Diego.
5. **Guardar só o `order_id` e renderizar no envio:** descartado em [09:52] Larissa.

## Questões para a ADR

- O estado "falhou" listado para a outbox em [09:08] convive com a tabela de DLQ separada decidida em [09:18]? O que significa "falhou" na outbox? [NEEDS INPUT: Diego]
- Qual o custo aceitável, em latência, da leitura de configuração e da escrita extra dentro de `changeStatus`? A reunião não definiu número.
- A criação do pedido (`null -> PENDING`, em `create`, fora de `changeStatus`) gera evento? A reunião não tratou.
- O que acontece com eventos já enfileirados quando o webhook é desativado ou removido? A reunião não tratou.
- Data da decisão. [NEEDS INPUT: a transcrição informa só "quinta-feira, 09:00"]

## Potential ADRs relacionados
- [Worker em processo separado com polling](worker-em-processo-separado-com-polling.md): consome a outbox.
- [Entrega at-least-once com X-Event-Id](entrega-at-least-once-com-x-event-id.md): o `event_id` nasce na inserção.
- [Reuso dos padrões do projeto](reuso-dos-padroes-do-projeto.md): UUID e integração via função que recebe a transação.

## Notas adicionais
Na iteração 0 o snapshot e o filtro viraram uma ADR própria. Pela Red Flag 5 do plugin (componente de uma decisão maior), foram consolidados aqui.
