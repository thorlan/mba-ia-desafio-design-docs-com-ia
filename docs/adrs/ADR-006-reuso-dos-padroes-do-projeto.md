# ADR-006: Reuso dos padrões existentes do projeto

**Status:** Aceito
**Data:** Reunião técnica de quinta-feira, 09:00 (ver `TRANSCRICAO.md`)
**ADRs relacionadas:** [ADR-001](ADR-001-outbox-transacional-no-mysql.md), [ADR-004](ADR-004-autenticacao-hmac-sha256-com-secret-por-endpoint.md)

---

## Contexto e Problema

O OMS já tem convenções consolidadas ([09:27] Bruno):

- Cada domínio é um módulo com controller, service, repository, routes e schemas, composto manualmente na inicialização da aplicação e publicado sob um prefixo de versão da API.
- Os erros de domínio derivam de `AppError`, com códigos em caixa alta, como `INSUFFICIENT_STOCK` e `INVALID_STATUS_TRANSITION` ([09:28] Bruno).
- Um error middleware centralizado converte `AppError`, erros do Zod e erros conhecidos do Prisma num envelope único.
- Os logs usam Pino, e o acesso é controlado por autenticação JWT e pelo `requireRole`.

A feature de webhooks traz um módulo novo, um segundo processo e um ponto de integração com pedidos. O problema é decidir se ela segue essas convenções ou ganha estrutura e bibliotecas próprias.

## Fatores de Decisão

- Manter respostas de erro e logs consistentes em toda a API ([09:29] Bruno).
- Não introduzir bibliotecas novas ([09:29] Bruno: "Não vamos botar nada novo").
- Caber no prazo de três sprints, com revisão de segurança ([09:46] Larissa).
- Qualquer pessoa do time conseguir trabalhar no módulo sem aprender um estilo novo.
- Acoplar o mínimo possível os módulos de pedidos e de webhooks ([09:41] Diego).

## Alternativas Consideradas

1. **Reuso máximo dos padrões existentes.**
2. **Convenções e bibliotecas próprias para o módulo.**
3. **Integração com pedidos por injeção de um repositório de webhooks** (variante de acoplamento avaliada na reunião).

## Decisão

Alternativa escolhida: **reuso máximo do que já existe** ([09:30] Larissa). O módulo de webhooks segue a mesma estrutura dos demais ([09:27] Bruno). Os erros derivam de `AppError`, com o prefixo `WEBHOOK_` em todos os códigos do módulo ([09:28] Bruno, [09:29] Larissa). O logger é o Pino já existente, e o error middleware não muda ([09:29] Bruno). As entradas são validadas com Zod ([09:30] Larissa). A operação administrativa usa o `requireRole` existente ([09:36] Larissa). Os identificadores seguem o padrão UUID ([09:51] Larissa).

A integração com pedidos é feita por uma função que recebe a transação corrente, em vez de injetar um repositório do módulo de webhooks no serviço de pedidos ([09:41] Bruno, [09:41] Diego).

## Prós e Contras das Alternativas

### Reuso máximo dos padrões existentes
- Pró: o mesmo envelope de erro, os mesmos logs e a mesma estrutura de pastas em toda a API.
- Pró: nenhuma mudança no error middleware, o que reduz a superfície de revisão.
- Contra: herda as limitações das classes e dos middlewares atuais.

### Convenções e bibliotecas próprias
- Pró: liberdade para modelar o módulo sob medida.
- Contra: duplica infraestrutura transversal e quebra a consistência das respostas.
- Contra: vai contra a diretriz explícita de não introduzir nada novo ([09:29] Bruno).

### Injeção de um repositório de webhooks no serviço de pedidos
- Pró: segue o estilo de injeção por construtor já usado na composição.
- Contra: acopla o serviço de pedidos a um repositório inteiro de outro módulo.
- Contra: desnecessário, porque "função pura recebendo o tx" resolve ([09:41] Diego).

## Consequências

**Positivas.** O módulo novo fala a mesma língua do resto da API: quem conhece um módulo conhece o de webhooks. Os erros do módulo chegam ao cliente no mesmo envelope, sem mudar o error middleware. E a integração com pedidos se limita a uma chamada dentro da transação existente.

**Negativas.** As limitações herdadas aparecem na implementação:
- Algumas classes de erro existentes fixam o código (por exemplo, a de recurso não encontrado sempre responde `NOT_FOUND`). Códigos como `WEBHOOK_NOT_FOUND` exigem estender `AppError` diretamente, como já fazem as classes de domínio.
- O middleware de validação converte todo erro do Zod em `VALIDATION_ERROR`, então uma regra validada só no schema não produz código `WEBHOOK_`.
- O error middleware atende apenas requisições HTTP, por isso o worker trata e registra os próprios erros.
- Os logs do worker saem com o mesmo nome de serviço da API.

**Trade-off explícito:** priorizamos consistência e velocidade de entrega sobre um desenho sob medida, aceitando as limitações das classes e dos middlewares existentes.

## Referências

- `src/shared/errors/app-error.ts:3` (classe base dos erros com código)
- `src/shared/errors/http-errors.ts:27` (classe de recurso não encontrado com código fixo; as linhas 33 e 39 aceitam código customizado)
- `src/middlewares/error.middleware.ts:14` (error middleware centralizado)
- `src/middlewares/validate.middleware.ts:31` (erros do Zod convertidos em `VALIDATION_ERROR`)
- `src/app.ts:26` (composição manual dos módulos; o serviço de pedidos é criado na linha 43)
- Transcrição: [09:27] Bruno, [09:28] Bruno, [09:29] Larissa, [09:29] Bruno, [09:30] Larissa, [09:36] Larissa, [09:41] Bruno, [09:41] Diego, [09:46] Larissa, [09:51] Larissa
