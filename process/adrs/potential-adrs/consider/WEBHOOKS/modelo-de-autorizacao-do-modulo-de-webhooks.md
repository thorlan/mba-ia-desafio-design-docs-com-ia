# Potential ADR: Modelo de autorização do módulo de webhooks

**Módulo:** WEBHOOKS (toca AUTH)
**Categoria:** Segurança
**Prioridade:** Consider (Score: 95)
**Data de identificação:** 25-09-2026

---

## O que foi identificado

A reunião definiu três regras de acesso ao módulo:

1. O **CRUD de configuração** de webhook exige apenas autenticação, **com qualquer role**, "por enquanto".
2. O **replay da DLQ** exige role `ADMIN`, reaproveitando o `requireRole`, e registra quem o executou.
3. O **`customer_id` é informado no body ou no path**, e não extraído do JWT. O JWT atual é do usuário operador, não do cliente.

O código mostra duas consequências que a reunião não discutiu:
- **Não existe vínculo entre `User` e `Customer`.** Qualquer usuário autenticado pode gerenciar webhooks de qualquer customer.
- **O registro de usuários é público e aceita `role: ADMIN`.** Isso esvazia a proteção do replay.

## Por que merece uma ADR (e por que está em "consider")

- **Impacto:** define quem pode ver e alterar para onde os dados de pedidos de cada cliente são enviados.
- **Trade-offs:** entrega rápida reaproveitando o modelo de roles atual, em troca de não isolar os dados por cliente.
- **Por que "consider":** a decisão foi fechada ([09:36] Larissa), mas é explicitamente provisória ([09:37] Sofia: "Por enquanto sim. Mais pra frente a gente pode endurecer"). Por isso atende mal ao critério "Estável" da regra dos 3 E's. Além disso, o custo de mudar a role exigida é baixo. É zona cinzenta, e cabe decisão humana: virar ADR própria, entrar como consequência na ADR de reuso de padrões, ou ir só para o FDD como riscos.

### Pontuação
| Dimensão | Nota | Justificativa |
| --- | --- | --- |
| Base (Step 0 adaptado) | 70 | Fechada ([09:36] Larissa: "Decidido, role ADMIN obrigatório no replay"), infraestrutura crítica do domínio (autorização) |
| Escopo e impacto | 10 | WEBHOOKS e o middleware de AUTH |
| Custo de mudança | 5 | Trocar a role exigida numa rota: menos de 1 semana |
| Conhecimento do time | 10 | Relevante ocasionalmente, para quem criar rotas no módulo |
| **Total** | **95** | "Estável" atendido só em parte (decisão declarada provisória) |

## Evidências

### Na transcrição
- [09:31] Marcos: "Customer_id implícito do JWT."
- [09:32] Bruno: "o JWT atual é do usuário operador, não do cliente."
- [09:32] Marcos: "A gente tem usuários que representam o cliente."
- [09:32] Larissa: "Então é endpoint autenticado normal, e o customer_id é passado no body ou no path. Não vem do JWT."
- [09:36] Sofia: "Mexer em fila de entrega de notificação não é coisa de operador. E o endpoint de admin tem que logar quem fez o replay"
- [09:36] Larissa: "Decidido, role ADMIN obrigatório no replay e a gente reaproveita o requireRole que já existe."
- [09:37] Sofia: sobre o CRUD com qualquer role, "Por enquanto sim. Mais pra frente a gente pode endurecer."

### No código
- [`src/middlewares/auth.middleware.ts`](../../../../../src/middlewares/auth.middleware.ts), linhas 19-25 e 49-61: o JWT carrega só `sub`, `email` e `role`; e o `requireRole`.
- [`prisma/schema.prisma`](../../../../../prisma/schema.prisma), linhas 10-13 e 25-54: enum `UserRole { ADMIN, OPERATOR }`; `User` e `Customer` sem relação.
- [`src/modules/auth/auth.routes.ts`](../../../../../src/modules/auth/auth.routes.ts), linha 10: `POST /register` sem `authenticate`.
- [`src/modules/auth/auth.schemas.ts`](../../../../../src/modules/auth/auth.schemas.ts), linha 7: `role: z.enum(['ADMIN', 'OPERATOR'])` aceito no body do registro.

### Linha do tempo na reunião
- **[09:31] a [09:32]:** o `customer_id` sai do JWT e passa a ser informado no body ou no path.
- **[09:35] a [09:36]:** ADMIN no replay e auditoria.
- **[09:36] a [09:37]:** CRUD com qualquer role, "por enquanto".
- **[09:48]:** o resumo final ("Endpoints CRUD de configuração autenticados normal, endpoint de replay de DLQ exige role ADMIN").

### Alternativas observadas
1. **`customer_id` extraído do JWT:** proposto em [09:31] Marcos, descartado em [09:32] Larissa.
2. **Restringir o CRUD por role:** adiado em [09:37] Sofia ("Mais pra frente").

## Questões para a ADR

- O risco de um usuário gerenciar webhooks de customers que não são dele é aceito nesta fase? [NEEDS INPUT: Sofia]
- O registro público com `role: ADMIN` é um risco que já existe no sistema. Ele fica registrado como risco do replay ou é tratado antes do deploy?

## Potential ADRs relacionados
- [Retry com backoff exponencial e DLQ](../../must-document/WEBHOOKS/retry-com-backoff-exponencial-e-dlq.md): o replay protegido.
- [Reuso dos padrões do projeto](../../must-document/WEBHOOKS/reuso-dos-padroes-do-projeto.md): o `requireRole` reaproveitado.

## Notas adicionais
Mudar o registro de usuários ou o modelo `User`/`Customer` está fora do escopo do desafio, que proíbe alterar o código. Aqui os dois pontos são só documentados como riscos.
