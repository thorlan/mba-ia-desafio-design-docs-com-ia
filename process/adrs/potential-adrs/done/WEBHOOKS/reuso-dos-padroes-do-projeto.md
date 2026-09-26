# Potential ADR: Reuso dos padrões existentes do projeto

**Módulo:** WEBHOOKS (toca PLATFORM)
**Categoria:** Framework / Plataforma
**Prioridade:** Must Document (Score: 125)
**Data de identificação:** 25-09-2026

---

## O que foi identificado

O módulo de webhooks segue as convenções que o projeto já tem, sem introduzir biblioteca ou estrutura nova:

- **Estrutura:** módulo em `src/modules/webhooks`, com controller, service, repository, routes e schemas.
- **Erros:** classes derivadas de `AppError`, com códigos de prefixo `WEBHOOK_`.
- **Logs:** o logger Pino já existente.
- **Tratamento de erros:** o error middleware centralizado, sem alteração.
- **Validação:** schemas Zod.
- **Autorização:** o `requireRole` já existente.
- **Identificadores:** UUID.

A integração com pedidos é feita por uma função que recebe a transação corrente, e não pela injeção de um repository de outro módulo no service de pedidos.

Consolidados aqui, por Red Flag 5: a estrutura de pastas, o prefixo de erro, o logger, o middleware, o UUID ([09:51] Larissa) e o `requireRole`.

## Por que merece uma ADR

- **Impacto:** define como o módulo conversa com toda a infraestrutura transversal (erros, logs, validação, autorização e composição).
- **Trade-offs:** consistência e velocidade, em troca de herdar as limitações das classes e middlewares atuais.
- **Complexidade:** baixa na intenção, mas com armadilhas concretas (ver Questões).
- **Conhecimento do time:** qualquer pessoa que trabalhe no módulo precisa conhecer essas convenções.
- **Implicações futuras:** o módulo evolui junto com os padrões do monólito.

### Pontuação
| Dimensão | Nota | Justificativa |
| --- | --- | --- |
| Base (Step 0 adaptado) | 75 | Fechada como decisão ([09:30] Larissa: "Decisão: reuso máximo do que já existe"), categoria framework e plataforma (estrutura da aplicação) |
| Escopo e impacto | 20 | Infraestrutura central: erros, logger, middlewares, composição e rotas |
| Custo de mudança | 10 | Trocar convenções do módulo: 1 a 2 semanas |
| Conhecimento do time | 20 | Crítico para qualquer trabalho no módulo |
| **Total** | **125** | Regra dos 3 E's atendida |

## Evidências

### Na transcrição
- [09:27] Bruno: "Cada domínio é um módulo em src/modules com controller, service, repository, routes e schemas. Webhook vai seguir igual."
- [09:28] Bruno: "Tem classe AppError, classes específicas tipo InsufficientStockError, InvalidStatusTransitionError. [...] Códigos tipo WEBHOOK_NOT_FOUND, WEBHOOK_INVALID_URL, WEBHOOK_SECRET_REQUIRED"
- [09:29] Larissa: "Prefixo WEBHOOK_ pra tudo do módulo."
- [09:29] Bruno: "o logger, que é Pino, já tá no projeto inteiro. Não vamos botar nada novo. O middleware de erro centralizado já trata AppError, Zod e Prisma."
- [09:30] Larissa: "Decisão: reuso máximo do que já existe. AppError, Pino, error middleware, padrão de módulos, padrão de schemas Zod, padrão de códigos de erro."
- [09:36] Larissa: "a gente reaproveita o requireRole que já existe."
- [09:41] Diego: "função pura recebendo o tx. Não precisa injetar repository inteiro."
- [09:51] Larissa: "UUID, segue o padrão do resto do projeto. Tudo é uuid."

### No código
- [`src/app.ts`](../../../../../src/app.ts), linhas 26-53 e 67: composição manual em `buildControllers` e montagem sob `/api/v1`. `OrderService` é criado com `(orderRepository, prisma)` na linha 43.
- [`src/shared/errors/http-errors.ts`](../../../../../src/shared/errors/http-errors.ts): `ConflictError` (33) e `UnprocessableEntityError` (39) aceitam `code`; `NotFoundError` (27) e `ValidationError` (9) fixam o código.
- [`src/middlewares/error.middleware.ts`](../../../../../src/middlewares/error.middleware.ts), linhas 14-65: tratamento de `AppError`, `ZodError`, `P2002` e `P2025`, com envelope `{ error: { code, message, details? } }`.
- [`src/middlewares/validate.middleware.ts`](../../../../../src/middlewares/validate.middleware.ts), linha 31: todo `ZodError` vira `ValidationError` (`VALIDATION_ERROR`).
- [`src/shared/logger/index.ts`](../../../../../src/shared/logger/index.ts), linha 20: `base: { service: 'order-management-api' }`.

### Linha do tempo na reunião
- **[09:27] a [09:30]:** a estrutura, os erros, o logger, o middleware e o fechamento.
- **[09:36]:** o `requireRole`.
- **[09:41]:** a função que recebe a transação, em vez de injetar um repository.
- **[09:48]:** o resumo final.
- **[09:51]:** o UUID.

### Alternativas observadas
1. **Introduzir convenções ou bibliotecas próprias do módulo:** descartado em [09:29] Bruno ("Não vamos botar nada novo") e em [09:30] Larissa.
2. **Injetar um repository de webhooks no `OrderService`:** descartado em [09:41] Diego.

## Questões para a ADR

- Uma URL `http` validada só no schema Zod devolve `VALIDATION_ERROR`, não `WEBHOOK_INVALID_URL` (`validate.middleware.ts:31`). Onde cada regra vive para os códigos `WEBHOOK_*` aparecerem de fato?
- `NotFoundError` não aceita código customizado. `WEBHOOK_NOT_FOUND` exige estender `AppError` diretamente, como fazem as classes de domínio existentes.
- O error middleware só atende requisições HTTP. O worker trata e loga os próprios erros, então o "vai pegar nossos erros sem precisar mudar nada" de [09:29] Bruno vale só para a API.
- O worker loga com `service: 'order-management-api'`. É preciso distinguir os logs dos dois processos?

## Potential ADRs relacionados
- [Outbox transacional no MySQL](outbox-transacional-no-mysql.md): integração via função com transação.
- [Autenticação HMAC-SHA256](autenticacao-hmac-sha256-com-secret-por-endpoint.md): o redaction do logger precisa cobrir `secret`.
- [Modelo de autorização do módulo](../../consider/WEBHOOKS/modelo-de-autorizacao-do-modulo-de-webhooks.md): uso do `requireRole`.
