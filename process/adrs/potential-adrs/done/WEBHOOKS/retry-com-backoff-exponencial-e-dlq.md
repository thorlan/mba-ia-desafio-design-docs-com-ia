# Potential ADR: Retry com backoff exponencial e DLQ em tabela separada

**Módulo:** WEBHOOKS (toca DATA e PLATFORM)
**Categoria:** Confiabilidade / Resiliência
**Prioridade:** Must Document (Score: 105)
**Data de identificação:** 25-09-2026

---

## O que foi identificado

Quando a entrega falha, o evento é retentado com **backoff exponencial** e um teto de tentativas: 1 min, 5 min, 30 min, 2 h e 12 h, somando quase 15 horas entre a primeira falha e a última tentativa. Uma resposta que não chega em 10 s conta como falha. Esgotadas as tentativas, o evento vai para uma **Dead Letter Queue em tabela separada** (`webhook_dead_letter`), com payload, motivo da falha e timestamp.

O reprocessamento da DLQ é **manual**, por um endpoint administrativo que devolve o evento à outbox como pendente. O endpoint exige role `ADMIN` e registra quem fez o replay.

Foram consolidados aqui, por Red Flag 3 e 5: o número de tentativas, a progressão, o timeout como critério de falha, a DLQ como tabela e o endpoint de replay.

## Por que merece uma ADR

- **Impacto:** define por quanto tempo um evento tenta ser entregue e quando exige intervenção humana. É parte do contrato implícito com o cliente.
- **Trade-offs:** cobre indisponibilidades longas sem eventos pendurados para sempre, em troca de entregas até ~15 h atrasadas e reprocessamento manual.
- **Complexidade:** média. Envolve agendar a próxima tentativa, contar tentativas, mover para outra tabela e fazer o replay.
- **Conhecimento do time:** suporte e operação precisam saber ler a DLQ e quando fazer replay.
- **Implicações futuras:** o aviso proativo ao cliente com falhas (e-mail) foi adiado. Hoje a DLQ só é vista por quem consulta.

### Pontuação
| Dimensão | Nota | Justificativa |
| --- | --- | --- |
| Base (Step 0 adaptado) | 70 | Fechada como decisão ([09:17] Larissa: "Decidido: 5 tentativas, backoff 1m/5m/30m/2h/12h"), infraestrutura crítica do domínio (entrega de notificações) |
| Escopo e impacto | 10 | WEBHOOKS e DATA |
| Custo de mudança | 10 | Mudar a política ou a DLQ: 1 a 2 semanas |
| Conhecimento do time | 15 | Importante para operação e suporte |
| **Total** | **105** | Regra dos 3 E's atendida |

## Evidências

### Na transcrição
- [09:15] Diego: "Backoff exponencial [...] depois de um teto de tentativas considera falha permanente e move pra DLQ."
- [09:15] Diego: "retry indefinido [...] traz o problema de evento ficar pendurado pra sempre se o cliente sumiu."
- [09:16] Diego: "3 é pouco. [...] retentaria três vezes em 30 minutos e mataria. Já tinha cliente nosso com indisponibilidade de duas horas"
- [09:17] Diego: "1 minuto, 5 minutos, 30 minutos, 2 horas, 12 horas. Total de quase 15 horas entre primeira falha e última tentativa."
- [09:17] Marcos: "Se um cliente meu cair por 15 horas, ele já tá com problema sério dele."
- [09:17] Larissa: "Faz numa tabela separada ou marca como 'failed' na própria outbox?"
- [09:18] Diego: "uma tabela webhook_dead_letter separada, com a payload, motivo da falha e timestamp. Mais limpa a leitura da outbox principal"
- [09:18] Diego: "Manual via endpoint admin. Tipo um POST /admin/webhooks/dead-letter/:id/replay. Recoloca na outbox como pendente."
- [09:36] Sofia: "Tem que ser ADMIN sim. [...] E o endpoint de admin tem que logar quem fez o replay, pra auditoria."
- [09:36] Larissa: "Decidido, role ADMIN obrigatório no replay e a gente reaproveita o requireRole que já existe."
- [09:42] Diego: "10 segundos. Cliente lento que não responde em 10s a gente trata como falha e marca pra retry."

### No código
- [`src/middlewares/auth.middleware.ts`](../../../../../src/middlewares/auth.middleware.ts), linhas 49-61: `requireRole`, reaproveitado no replay.
- [`src/modules/users/user.routes.ts`](../../../../../src/modules/users/user.routes.ts), linha 15: único uso atual de `requireRole('ADMIN')`, que serve de modelo.
- [`src/app.ts`](../../../../../src/app.ts), linha 67: rotas montadas sob `/api/v1`, então o replay fica em `/api/v1/admin/webhooks/dead-letter/:id/replay`.
- [`src/shared/logger/index.ts`](../../../../../src/shared/logger/index.ts): Pino, onde o log de auditoria do replay é gravado.

### Linha do tempo na reunião
- **[09:14] a [09:17]:** retry, número de tentativas e progressão, fechados em [09:17] Larissa.
- **[09:17] a [09:18]:** DLQ em tabela separada e replay manual.
- **[09:35] a [09:36]:** role `ADMIN` e auditoria.
- **[09:42]:** o timeout como critério de falha.
- **[09:48]:** o resumo final.

### Alternativas observadas
1. **Retry indefinido com backoff:** descartado em [09:15] Diego.
2. **Apenas 3 tentativas:** proposto em [09:16] Bruno, descartado em [09:16] Diego.
3. **Marcar como "failed" na própria outbox:** levantado em [09:17] Larissa, descartado em [09:18] Diego a favor da tabela separada.

## Questões para a ADR

- "5 tentativas" são 5 chamadas no total ou 5 retentativas após o envio inicial? Os 5 intervalos somam 14h36, o que bate com "quase 15 horas" ([09:17] Diego). O exemplo de "três vezes em 30 minutos" (1 + 5 + 30 min, [09:16] Diego) também indica retentativas **após** o envio inicial, ou seja, 6 chamadas no máximo. [NEEDS INPUT: confirmar com Diego]
- Quais respostas contam como falha? Só a ausência de resposta foi definida ([09:42] Diego). Respostas 4xx também devem ser retentadas?
- O replay mantém o `event_id` original ou gera um novo? Isso afeta a deduplicação do cliente.
- O log de auditoria do replay fica só no Pino ou também é persistido? A reunião diz apenas "logar" ([09:36] Sofia).

## Potential ADRs relacionados
- [Worker em processo separado com polling](worker-em-processo-separado-com-polling.md): executa as tentativas.
- [Entrega at-least-once com X-Event-Id](entrega-at-least-once-com-x-event-id.md): o retry é a origem das duplicatas.
- [Modelo de autorização do módulo](../../consider/WEBHOOKS/modelo-de-autorizacao-do-modulo-de-webhooks.md): a role exigida no replay.

## Notas adicionais
O e-mail para avisar o cliente de falhas seguidas foi explicitamente adiado para a próxima fase ([09:37] Larissa) e não entra como consequência resolvida.
