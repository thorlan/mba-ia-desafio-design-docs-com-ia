# Potential ADR: Entrega at-least-once com X-Event-Id para deduplicação

**Módulo:** WEBHOOKS
**Categoria:** Arquitetura / Contrato de integração
**Prioridade:** Must Document (Score: 125)
**Data de identificação:** 25-09-2026

---

## O que foi identificado

A plataforma garante entrega **at-least-once**: o mesmo evento pode chegar mais de uma vez, e o cliente precisa estar preparado. Cada evento recebe um **UUID gerado quando entra na outbox**, único por evento, enviado no header **`X-Event-Id`** e também no campo `event_id` do payload. O cliente deduplica por esse identificador.

A alternativa exactly-once foi descartada por exigir coordenação dos dois lados. A responsabilidade transferida ao cliente foi reconhecida e compensada com documentação destacada no portal do desenvolvedor.

## Por que merece uma ADR

- **Impacto:** define a semântica de entrega que todo cliente precisa tratar. É parte do contrato externo.
- **Trade-offs:** plataforma simples e sem perda de eventos, em troca de duplicatas eventuais e de responsabilidade no cliente.
- **Complexidade:** baixa na plataforma e variável no cliente.
- **Conhecimento do time:** suporte, produto e quem mexer no envio precisam conhecer a garantia oferecida.
- **Implicações futuras:** trocar a garantia muda o contrato com todos os clientes.

### Pontuação
| Dimensão | Nota | Justificativa |
| --- | --- | --- |
| Base (Step 0 adaptado) | 70 | Fechada como decisão ([09:26] Larissa: "At-least-once com X-Event-Id pra dedup do lado do cliente. Decisão."), infraestrutura crítica do domínio (entrega de notificações) |
| Escopo e impacto | 20 | Afeta todas as integrações externas |
| Custo de mudança | 20 | Mudar a garantia exige renegociar o contrato com os clientes: 2 a 6 meses |
| Conhecimento do time | 15 | Importante para o módulo, o suporte e o produto |
| **Total** | **125** | Regra dos 3 E's atendida |

## Evidências

### Na transcrição
- [09:24] Diego: "a gente vai garantir at-least-once. Pode acontecer de o cliente receber o mesmo evento duas vezes. Ele tem que estar preparado."
- [09:25] Diego: "A gente manda um event_id no header, X-Event-Id, com um UUID gerado quando o evento entra na outbox. É único por evento."
- [09:25] Sofia: "Isso joga responsabilidade pro cliente."
- [09:25] Diego: "Stripe faz assim, GitHub faz assim. Garantir exactly-once exigiria coordenação dos dois lados [...] At-least-once com event_id resolve 99% dos casos."
- [09:26] Marcos: "Eu posso documentar isso bem destacado no portal de desenvolvedor pros clientes"
- [09:26] Larissa: "At-least-once com X-Event-Id pra dedup do lado do cliente. Decisão."
- [09:43] Diego: payload com `event_id` entre os campos

### No código
- [`src/middlewares/request-logger.middleware.ts`](../../../../../src/middlewares/request-logger.middleware.ts), linha 2: `uuid` v4 já é usado para gerar o `X-Request-Id`. A biblioteca está disponível para o `event_id`.
- [`prisma/schema.prisma`](../../../../../prisma/schema.prisma), linha 26: padrão `@default(uuid()) @db.Char(36)` para identificadores.

### Linha do tempo na reunião
- **[09:24] a [09:26]:** a garantia, o `X-Event-Id`, a objeção de Sofia, o descarte de exactly-once e o fechamento.
- **[09:43] a [09:44]:** o `event_id` no payload e no header.
- **[09:48]:** o resumo final ("Idempotência por X-Event-Id, garantia at-least-once").

### Alternativas observadas
1. **Exactly-once:** descartado em [09:25] Diego (coordenação dos dois lados).
2. **Deduplicação garantida pela plataforma, sem ônus para o cliente:** a preocupação foi levantada em [09:25] Sofia, e a plataforma não tem como saber se o cliente processou uma requisição que estourou o timeout.

## Questões para a ADR

- Num replay da DLQ, o evento mantém o `event_id` original ou recebe um novo? Manter preserva a deduplicação; gerar um novo força o reprocessamento. [NEEDS INPUT: Diego]
- Quando o mesmo evento vai para dois webhooks do mesmo customer, o `event_id` é o mesmo nos dois ou é um por entrega? O `X-Webhook-Id` ([09:44] Sofia) distingue o destino, mas a unicidade "por evento" ([09:25] Diego) não define esse caso.

## Potential ADRs relacionados
- [Outbox transacional no MySQL](outbox-transacional-no-mysql.md): o `event_id` nasce na inserção.
- [Retry com backoff exponencial e DLQ](retry-com-backoff-exponencial-e-dlq.md): origem das duplicatas e do replay.
- [Autenticação HMAC-SHA256](autenticacao-hmac-sha256-com-secret-por-endpoint.md): o outro header do contrato de entrega.

## Notas adicionais
O comentário de Marcos sobre o portal ([09:26]) é uma ação de produto que vira dependência no PRD, não parte da decisão técnica.
