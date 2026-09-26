# Potential ADR: Autenticação HMAC-SHA256 com secret por endpoint e rotação

**Módulo:** WEBHOOKS (toca PLATFORM)
**Categoria:** Segurança
**Prioridade:** Must Document (Score: 125)
**Data de identificação:** 25-09-2026

---

## O que foi identificado

Cada entrega é assinada com **HMAC-SHA256 sobre o corpo do request**, e a assinatura vai no header `X-Signature`. Assim o cliente verifica que a requisição veio da plataforma e que o payload não foi adulterado.

A secret é **única por endpoint de webhook**, nunca global. É **gerada pela plataforma** e devolvida ao cliente na criação do webhook. Ela é **rotacionável** pela API, e a secret anterior continua válida por **24 horas** em paralelo, para o cliente migrar sem janela de falha.

Consolidados aqui, por Red Flag 5: a secret por endpoint, a geração pela plataforma, a rotação e a carência. A exigência de `https` foi declarada pela própria reunião como "nem é decisão arquitetural" ([09:23] Sofia) e vai para o FDD como validação.

## Por que merece uma ADR

- **Impacto:** é o mecanismo de confiança entre a plataforma e todos os clientes integrados. Todo cliente implementa a verificação do lado dele.
- **Trade-offs:** padrão de mercado e raio de vazamento limitado a um endpoint, em troca de gerenciar secrets recuperáveis, rotação e carência.
- **Complexidade:** média. A geração segura da secret, o armazenamento, a rotação com duas secrets válidas e a revisão de segurança dedicada.
- **Conhecimento do time:** quem mexer no envio ou no cadastro precisa entender as regras da secret.
- **Implicações futuras:** mudar o esquema de assinatura obriga todos os clientes a mudar a integração.

### Pontuação
| Dimensão | Nota | Justificativa |
| --- | --- | --- |
| Base (Step 0 adaptado) | 70 | Fechada como decisão ([09:22] Sofia: "Decidido: HMAC-SHA256 sobre o corpo do request, secret por endpoint, suporte a rotação com grace period de 24h"), infraestrutura crítica do domínio (autenticação) |
| Escopo e impacto | 20 | Todos os endpoints de webhook e todas as integrações externas |
| Custo de mudança | 20 | Trocar o esquema exige coordenar todos os clientes: 2 a 6 meses |
| Conhecimento do time | 15 | Importante para o módulo e para a revisão de segurança |
| **Total** | **125** | Regra dos 3 E's atendida |

## Evidências

### Na transcrição
- [09:19] Sofia: "O cliente tem que conseguir validar que a requisição veio realmente da gente, e que ninguém adulterou o payload no meio."
- [09:20] Sofia: "Padrão é HMAC. A gente assina o payload com uma secret compartilhada [...] manda a assinatura num header tipo X-Signature."
- [09:20] Sofia: "SHA-256. HMAC-SHA256 é o padrão de mercado, todo cliente sério tem biblioteca pra isso."
- [09:21] Sofia: "cada endpoint de webhook do cliente tem que ter uma secret única. Não é uma secret global da nossa plataforma. Senão se vaza uma, vaza tudo."
- [09:21] Bruno: "a tabela de configuração de webhook armazena url + secret + customer_id + estado ativo?" ([09:21] Sofia: "Sim.")
- [09:21] Sofia: "Quando ele rotaciona, a antiga fica válida por 24 horas em paralelo [...] Depois disso, a antiga morre."
- [09:22] Diego: "A gente já teve cliente que vazou secret em log de aplicação dele uma vez."
- [09:22] Sofia: "Decidido: HMAC-SHA256 sobre o corpo do request, secret por endpoint, suporte a rotação com grace period de 24h."
- [09:23] Sofia: "TLS obrigatório. [...] Isso na verdade nem é decisão arquitetural, é só uma validação no schema Zod."
- [09:31] Marcos: "secret é gerada pela gente e devolvida na criação."
- [09:46] Sofia: "Reservem pelo menos dois dias úteis pra eu revisar o código de segurança antes do deploy. HMAC e geração de secret eu quero olhar com calma."

### No código
- [`src/modules/users/user.service.ts`](../../../../../src/modules/users/user.service.ts), linhas 16 e 26: senhas guardadas como hash bcrypt. A secret de webhook **não pode** seguir esse padrão, porque o worker precisa dela em claro para assinar.
- [`src/shared/logger/index.ts`](../../../../../src/shared/logger/index.ts), linhas 4-11: `redactPaths` não inclui `secret`.
- [`src/middlewares/validate.middleware.ts`](../../../../../src/middlewares/validate.middleware.ts): onde a regra de `https` do schema Zod é aplicada.

### Linha do tempo na reunião
- **[09:19] a [09:22]:** o problema, o HMAC, o algoritmo, a secret por endpoint, a rotação e o fechamento.
- **[09:23]:** o TLS, classificado como validação.
- **[09:31]:** a secret gerada e devolvida na criação.
- **[09:44]:** os headers de envio.
- **[09:46]:** a revisão de segurança.
- **[09:48]:** o resumo final.

### Alternativas observadas
1. **Secret global da plataforma:** descartada em [09:21] Sofia.
2. **Aceitar URL `http`:** descartado em [09:23] Sofia, como validação e não como alternativa arquitetural.

## Questões para a ADR

- Durante a carência de 24 h, a mensagem é assinada com as duas secrets (dois valores em `X-Signature`) ou só com a nova? [NEEDS INPUT: Sofia]
- Como a secret é protegida em repouso, já que precisa ser recuperável? A reunião deixa isso para a revisão de segurança ([09:46] Sofia).
- O HMAC cobre só o corpo ([09:22] Sofia). O `X-Timestamp`, pensado para o cliente detectar replay ([09:44] Diego), não é assinado. A lacuna é aceita?
- A secret só é exibida na criação e na rotação, ou também pode ser consultada depois? A reunião cita apenas "devolvida na criação".

## Potential ADRs relacionados
- [Entrega at-least-once com X-Event-Id](entrega-at-least-once-com-x-event-id.md): o outro header do contrato de entrega.
- [Reuso dos padrões do projeto](reuso-dos-padroes-do-projeto.md): Zod e o logger com redaction.

## Notas adicionais
A exigência de `https` e o limite de 64 KB foram classificados pela própria reunião como validação e requisito não funcional ([09:23] Sofia, [09:24] Larissa). Não viram ADR.
