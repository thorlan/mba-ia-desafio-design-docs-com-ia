# ADR-006: Reuso dos padrões existentes do projeto

**Status:** Aceito
**Data:** Reunião técnica de quinta-feira, 09:00 (ver `TRANSCRICAO.md`)
**Relacionada a:**
- [ADR-001: Outbox transacional no MySQL existente](./ADR-001-outbox-transacional-no-mysql.md)
- [ADR-002: Worker em processo separado com polling](./ADR-002-worker-em-processo-separado-com-polling.md)
- [ADR-003: Retry com backoff exponencial e DLQ em tabela separada](./ADR-003-retry-com-backoff-exponencial-e-dlq.md)

## Contexto e Problema

O OMS já tem convenções consolidadas:

- Cada domínio é um módulo com controller, service, repository, routes e schemas ([09:27] Bruno; no código, `src/modules/auth/` não tem repository), composto manualmente na inicialização da aplicação e publicado sob um prefixo de versão da API (`src/app.ts:26`, `src/app.ts:67`).
- Os erros de domínio derivam de `AppError`, com códigos em caixa alta, como `INSUFFICIENT_STOCK` e `INVALID_STATUS_TRANSITION` ([09:28] Bruno).
- Um error middleware centralizado trata `AppError`, erros do Zod e do Prisma ([09:29] Bruno, `src/middlewares/error.middleware.ts:14`).
- Os logs usam Pino ([09:29] Bruno), e o acesso é controlado por autenticação JWT e pelo `requireRole` (`src/middlewares/auth.middleware.ts:27`, `src/middlewares/auth.middleware.ts:49`).

A feature de webhooks traz um módulo novo ([09:27] Bruno), um segundo processo ([09:28] Bruno) e um ponto de integração com pedidos ([09:40] Bruno). O problema é decidir se ela segue essas convenções ou ganha estrutura e bibliotecas próprias.

## Fatores de Decisão

- Manter respostas de erro e logs consistentes em toda a API ([09:29] Bruno).
- Não introduzir bibliotecas novas ([09:29] Bruno: "Não vamos botar nada novo").
- Acoplar o mínimo possível os módulos de pedidos e de webhooks ([09:41] Diego).

## Alternativas Consideradas

1. **Reuso máximo dos padrões existentes.**
2. **Convenções e bibliotecas próprias para o módulo.**

## Decisão

Alternativa escolhida: **reuso máximo do que já existe** ([09:30] Larissa), porque mantém erros e logs consistentes em toda a API sem introduzir nada novo ([09:29] Bruno). O módulo de webhooks segue a mesma estrutura dos demais ([09:27] Bruno). Os erros derivam de `AppError`, com o prefixo `WEBHOOK_` em todos os códigos do módulo ([09:28] Bruno, [09:29] Larissa). O logger é o Pino já existente, e o error middleware não muda ([09:29] Bruno). As entradas são validadas com Zod ([09:30] Larissa). A operação administrativa usa o `requireRole` existente ([09:36] Larissa). Os identificadores seguem o padrão UUID ([09:51] Larissa).

A integração com pedidos é feita por uma função que recebe a transação corrente, em vez de injetar um repositório do módulo de webhooks no serviço de pedidos ([09:41] Bruno, [09:41] Diego).

## Prós e Contras das Alternativas

### Reuso máximo dos padrões existentes
- Pró: mesma estrutura de módulo, mesmos erros e mesmo logger em toda a API ([09:27] Bruno, [09:29] Bruno).
- Pró: o error middleware pega os erros do módulo "sem precisar mudar nada" ([09:29] Bruno).
- Contra: herda as limitações das classes e dos middlewares atuais (`src/shared/errors/http-errors.ts:27`, `src/middlewares/validate.middleware.ts:31`).

### Convenções e bibliotecas próprias
- Contra: vai contra a diretriz explícita de não introduzir nada novo ([09:29] Bruno).

## Consequências

**Positivas.** O módulo novo segue a mesma estrutura dos demais ([09:27] Bruno). Os erros do módulo passam pelo error middleware sem mudança nele ([09:29] Bruno). E a integração com pedidos se limita a uma chamada dentro da transação existente ([09:41] Bruno).

**Negativas.** As limitações herdadas aparecem na implementação:
- Algumas classes de erro existentes fixam o código (por exemplo, a de recurso não encontrado sempre responde `NOT_FOUND`, `src/shared/errors/http-errors.ts:27-31`). Só a classe base e as classes HTTP de requisição inválida, conflito e entidade não processável aceitam código próprio, e é dessas duas últimas que derivam as classes de domínio. Um código como `WEBHOOK_NOT_FOUND` não sai da classe de recurso não encontrado atual.
- O middleware de validação converte todo erro do Zod em `VALIDATION_ERROR` (`src/middlewares/validate.middleware.ts:31`), então uma regra validada só no schema não produz código `WEBHOOK_`. Isso conflita com validar o https no schema ([09:23] Sofia) e ter o código `WEBHOOK_INVALID_URL` ([09:28] Bruno), e fica como questão em aberto no RFC.
- O error middleware é um middleware HTTP (`src/middlewares/error.middleware.ts:14`) e não passa pelos erros do worker, que não é HTTP.
- O logger tem um nome de serviço fixo (`src/shared/logger/index.ts:20`), então os logs do worker saem com o mesmo nome da API.

**Trade-off explícito:** priorizamos a consistência com o que já existe e não introduzir nada novo ([09:29] Bruno, [09:30] Larissa), aceitando as limitações das classes e dos middlewares atuais.

## Referências

- `src/shared/errors/app-error.ts:3` (classe base dos erros com código)
- `src/shared/errors/http-errors.ts:27` (classe de recurso não encontrado com código fixo; as linhas 3, 33 e 39 aceitam código customizado e as classes de domínio, nas linhas 45 e 55, derivam das duas últimas)
- `src/middlewares/error.middleware.ts:14` (error middleware centralizado)
- `src/middlewares/validate.middleware.ts:31` (erros do Zod convertidos em `VALIDATION_ERROR`)
- `src/app.ts:26` (composição manual dos módulos; o serviço de pedidos é criado na linha 43)
- Transcrição: [09:23] Sofia, [09:27] Bruno, [09:28] Bruno, [09:29] Bruno, [09:29] Larissa, [09:30] Larissa, [09:36] Larissa, [09:40] Bruno, [09:41] Diego, [09:41] Bruno, [09:51] Larissa
