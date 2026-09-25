# Mapeamento da arquitetura do código

> Fase 1 do fluxo de ADRs, gerada com o prompt `process/prompts/01-adr-map-adaptado.md`.
> Contexto utilizado: `TRANSCRICAO.md`.

## Visão geral do projeto

- **Nome:** `order-management-api` (`package.json`)
- **Propósito:** API REST de um Order Management System (OMS). Cobre autenticação, usuários, clientes, produtos e pedidos. O ciclo de vida do pedido segue uma máquina de estados, com controle transacional de estoque e histórico de mudanças de status.
- **Tipo:** monólito modular, um único processo HTTP (`src/server.ts`).
- **Linguagem:** TypeScript (ESM, `strict`), Node.js >= 20.
- **Framework:** Express 4.

## Stack tecnológica

| Camada | Tecnologia | Evidência |
| --- | --- | --- |
| Runtime | Node.js >= 20, TypeScript 5.6 | `package.json` (engines), `tsconfig.json` |
| HTTP | Express 4.21 | `src/app.ts` |
| Banco de dados | MySQL 8.0 (Docker) | `docker-compose.yml`, `prisma/schema.prisma:6` |
| ORM | Prisma 5.22 | `src/config/database.ts` |
| Validação | Zod 3.23 | `src/middlewares/validate.middleware.ts`, `src/config/env.ts` |
| Autenticação | JWT (`jsonwebtoken`), senhas com `bcrypt` | `src/middlewares/auth.middleware.ts`, `src/modules/users/user.service.ts:26` |
| Logs | Pino 9.5 (`pino-pretty` em desenvolvimento) | `src/shared/logger/index.ts` |
| Identificadores | UUID (`@default(uuid()) @db.Char(36)`); `uuid` v4 no `X-Request-Id` | `prisma/schema.prisma:26`, `src/middlewares/request-logger.middleware.ts:2` |
| Testes | Vitest + supertest contra MySQL real | `vitest.config.ts`, `tests/setup.ts` |

Não existe no projeto: fila, broker, cache, cliente HTTP de saída, agendador, biblioteca de métricas ou de tracing. A dependência `pino-http` está declarada em `package.json` mas não é importada em `src/`.

## Notas de contexto

**Fonte:** `TRANSCRICAO.md`, reunião técnica de ~55 min com Larissa (Tech Lead), Marcos (PM), Bruno (Pedidos), Diego (Plataforma, entra às 09:05) e Sofia (Segurança).

**Principais conclusões**
- **Feature:** webhooks de saída que notificam clientes B2B quando o status de um pedido muda. Latência aceitável abaixo de 10 s ([09:00] Marcos, [09:02] Marcos, [09:03] Sofia).
- **Padrões arquiteturais decididos:** outbox transacional no MySQL, worker em processo separado com polling, retry com backoff e DLQ, HMAC por endpoint, entrega at-least-once, reuso dos padrões do projeto (resumo em [09:48] Larissa).
- **Módulo novo:** `src/modules/webhooks` (a criar), no mesmo padrão dos demais ([09:27] Bruno).
- **Novo entry point:** `src/worker.ts` (a criar) e o script `npm run worker` ([09:11] Larissa).

**Discrepâncias entre a reunião e o código**
1. **Vínculo usuário/cliente.** [09:32] Marcos diz que "a gente tem usuários que representam o cliente", mas `prisma/schema.prisma` não tem relação entre `User` e `Customer`. As roles são apenas `ADMIN` e `OPERATOR`, e o JWT carrega só `sub`, `email` e `role` (`src/middlewares/auth.middleware.ts:19-25`). Não há como restringir um usuário aos webhooks do seu próprio customer.
2. **Registro de ADMIN.** O endpoint de replay exigirá role `ADMIN` ([09:36] Sofia), mas `POST /api/v1/auth/register` é público (`src/modules/auth/auth.routes.ts:10`) e aceita `role: 'ADMIN'` no body (`src/modules/auth/auth.schemas.ts:7`).
3. **Erros com prefixo.** [09:29] Bruno afirma que o error middleware "vai pegar nossos erros sem precisar mudar nada". Vale para a API. Mas o `validate.middleware.ts:31` converte todo `ZodError` em `ValidationError` (código `VALIDATION_ERROR`), então regras validadas só no schema não produzem códigos `WEBHOOK_*`. E o worker, que não é HTTP, não passa pelo middleware.
4. **Classes de erro.** Nem toda classe de `src/shared/errors/http-errors.ts` aceita código customizado. `NotFoundError` (linha 27) fixa `NOT_FOUND`, enquanto `ConflictError` (linha 33) e `UnprocessableEntityError` (linha 39) aceitam um `code`.
5. **Caminhos das rotas.** A transcrição cita rotas sem prefixo (`/admin/webhooks/...`, `/webhooks/:id/deliveries`), mas todas as rotas são montadas sob `/api/v1` (`src/app.ts:67`).
6. **Mascaramento de segredos.** O logger mascara `password`, `passwordHash`, `token` e `accessToken` (`src/shared/logger/index.ts:4-11`), mas não `secret`. A secret de webhook ([09:21] Sofia) ficaria exposta em log.
7. **Geração de eventos.** A criação do pedido grava o histórico `null -> PENDING` dentro de `create` (`src/modules/orders/order.service.ts:58`), e não em `changeStatus`. A reunião fala apenas de "quando o status muda" e não trata esse caso.

## Módulos do sistema

### Índice de módulos

1. **PLATFORM:** bootstrap, composição, rotas, configuração, middlewares e utilitários compartilhados
2. **DATA:** schema, migrations e seed do Prisma
3. **AUTH:** registro, login e usuário corrente
4. **USERS:** consulta de usuários (somente ADMIN)
5. **CUSTOMERS:** CRUD de clientes
6. **PRODUCTS:** CRUD de produtos
7. **ORDERS:** pedidos, máquina de estados, estoque e histórico
8. **WEBHOOKS (a criar):** a feature discutida na reunião

### PLATFORM: bootstrap e infraestrutura compartilhada
- **Propósito:** subir o processo HTTP, compor dependências, montar rotas e prover erros, logs, validação e autenticação.
- **Localização:** `src/server.ts`, `src/app.ts`, `src/routes/`, `src/config/`, `src/middlewares/`, `src/shared/` (14 arquivos)
- **Componentes-chave:**
  - `bootstrap()` com shutdown gracioso em `SIGINT` e `SIGTERM` (`src/server.ts:6-21`)
  - `buildControllers` monta as dependências à mão (`src/app.ts:26-53`)
  - `buildApiRouter` sob `/api/v1` (`src/routes/index.ts:21`, `src/app.ts:67`)
  - `createPrismaClient` e o singleton `prisma` (`src/config/database.ts:4-10`)
  - `env` validado por Zod na carga, com `process.exit(1)` se inválido (`src/config/env.ts`)
  - `AppError` com `statusCode`, `errorCode` e `details` (`src/shared/errors/app-error.ts:3`)
  - `errorMiddleware` para `AppError`, `ZodError`, `P2002` e `P2025` (`src/middlewares/error.middleware.ts:14-65`)
  - `authenticate` e `requireRole` (`src/middlewares/auth.middleware.ts:27, 49`)
  - `validate` (`src/middlewares/validate.middleware.ts:11`)
  - `requestLogger` com `X-Request-Id` (`src/middlewares/request-logger.middleware.ts`)
  - `paginated` (`src/shared/http/response.ts`)
  - `logger` Pino (`src/shared/logger/index.ts:32`)
- **Padrões:** injeção de dependência manual por construtor, erro de domínio como exceção tipada e envelope `{ error: { code, message, details? } }`.
- **Escopo:** médio.

### DATA: persistência
- **Propósito:** modelo relacional e evolução do schema.
- **Localização:** `prisma/schema.prisma`, `prisma/migrations/20260519182739_init/`, `prisma/seed.ts`
- **Componentes-chave:**
  - Modelos `User`, `Customer`, `Product`, `Order`, `OrderItem`, `OrderStatusHistory` e `OrderNumberSequence` (`prisma/schema.prisma:25-138`)
  - Enums `UserRole` e `OrderStatus`
- **Padrões:** UUID `CHAR(36)` em todas as entidades, índices em colunas de filtro e `onDelete: Cascade` de `Order` para itens e histórico.
- **Escopo:** pequeno.

### AUTH: autenticação
- **Localização:** `src/modules/auth/` (4 arquivos: controller, routes, schemas, service; sem repository, reusa `UserRepository`)
- **Componentes-chave:** `POST /auth/register` (público), `POST /auth/login` e `GET /auth/me`. O JWT é assinado com `sub`, `email` e `role`.
- **Observação:** o register aceita `role` no body (ver discrepância 2).

### USERS: usuários
- **Localização:** `src/modules/users/` (5 arquivos)
- **Componentes-chave:** `GET /users/:id` com `requireRole('ADMIN')` (`src/modules/users/user.routes.ts:15`). É o único uso atual de `requireRole`.

### CUSTOMERS e PRODUCTS: cadastros
- **Localização:** `src/modules/customers/` e `src/modules/products/` (5 arquivos cada)
- **Padrão:** CRUD completo com `authenticate` em todas as rotas, paginação e unicidade de e-mail ou SKU tratada como `ConflictError` com código de domínio (`EMAIL_ALREADY_USED`, `SKU_ALREADY_USED`).

### ORDERS: pedidos
- **Propósito:** criar, listar, consultar, mudar status e excluir pedidos.
- **Localização:** `src/modules/orders/` (6 arquivos)
- **Componentes-chave:**
  - Máquina de estados `PENDING -> PAID -> PROCESSING -> SHIPPED -> DELIVERED`, com `CANCELLED` a partir de `PENDING`, `PAID` e `PROCESSING` (`src/modules/orders/order.status.ts:3-10`)
  - `canTransition`, `shouldDebitStock` e `shouldReplenishStock` (`order.status.ts:12, 29, 33`)
  - `changeStatus` (`src/modules/orders/order.service.ts:126-179`): um único `prisma.$transaction` (linha 131) que lê o pedido com itens, valida a transição, debita ou repõe estoque, atualiza `orders` (linha 158) e insere em `order_status_history` (linha 159)
  - Tipo `TxClient = Prisma.TransactionClient` (linha 24), já usado por helpers internos que recebem a transação (linhas 205, 234 e 245)
  - Rota `PATCH /orders/:id/status` (`src/modules/orders/order.routes.ts`)
- **Relevância para a feature:** é o ponto de inserção do evento na outbox ([09:40] Bruno).

### WEBHOOKS (a criar): notificação de pedidos
Descrito somente a partir da transcrição.
- **Localização prevista:** `src/modules/webhooks/` com controller, service, repository, routes e schemas ([09:27] Bruno); `webhook.worker.ts` ou `webhook.processor.ts` para o processamento ([09:28] Bruno); `src/worker.ts` como entry point ([09:11] Larissa, [09:28] Bruno).
- **Dados previstos:**
  - Configuração de webhook com url, secret, customer_id, estado ativo e lista de status assinados ([09:21] Bruno, [09:33] Marcos)
  - `webhook_outbox` ([09:06] Diego)
  - `webhook_dead_letter` ([09:18] Diego)
  - Histórico de entregas ([09:34] Marcos)
- **Interfaces previstas:**
  - CRUD de configuração ([09:31] Marcos, [09:33] Bruno)
  - Rotação de secret ([09:21] Sofia)
  - `GET /webhooks/:id/deliveries` ([09:34] Marcos)
  - `POST /admin/webhooks/dead-letter/:id/replay` ([09:18] Diego)
  - Chamada HTTP de saída para o cliente com os headers `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type` ([09:44] Diego, [09:44] Sofia)
- **Dependências:** ORDERS (inserção do evento), PLATFORM (erros, logger, middlewares, Prisma) e DATA (novas tabelas).

## Aspectos transversais

| Aspecto | Hoje | Onde a feature toca |
| --- | --- | --- |
| Transações | `prisma.$transaction` interativo com `TxClient` | inserção na outbox dentro de `changeStatus` |
| Erros | `AppError` e subclasses, códigos em UPPER_SNAKE_CASE | novos códigos `WEBHOOK_*` |
| Autorização | `authenticate` e `requireRole('ADMIN')` | CRUD autenticado, replay com ADMIN |
| Validação | Zod via `validate` | URL `https`, filtro de status |
| Logs | Pino com redaction | auditoria do replay; campo `secret` fora da redaction |
| Processo | um único processo HTTP com shutdown gracioso | segundo processo (worker) |
| Configuração | `env.ts` valida todas as variáveis na carga | o worker herda a mesma validação (exige `JWT_SECRET`) |
| Testes | Vitest com limpeza de tabelas em `tests/setup.ts` | novas tabelas entram na limpeza |
