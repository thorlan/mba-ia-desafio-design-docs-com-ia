# Registro de iterações

Cada entrada registra o que a IA produziu, o que a revisão encontrou e o que mudou. Serve de base para a seção "Iterações e ajustes" do `README.md`.

## Etapa 1: exploração inicial

- **Pedido:** ler todo o repositório (enunciado, transcrição, `src/`, `prisma/`, `tests/`, configs) e fazer um estudo do que o sistema faz, do que não faz, do que pode e do que não pode.
- **Resultado útil:** além do mapa do código, a exploração levantou pontos que um documento gerado sem cuidado erraria. Por exemplo:
  - "5 tentativas" só fecha com "quase 15 horas" se forem 5 retentativas após o envio inicial.
  - O `customer_id` não vem do JWT: a proposta de [09:31] Marcos é corrigida em [09:32] Larissa.
  - Retentativas quebram a ordem por pedido.
  - O `validate.middleware.ts` transforma erros do Zod em `VALIDATION_ERROR`, o que conflita com `WEBHOOK_INVALID_URL`.
  - `POST /auth/register` é público e aceita `role: ADMIN`.

## Iteração 0: ADRs gerados direto da transcrição (descartados)

- **O que a IA fez:** logo após a exploração, escreveu 7 ADRs diretamente a partir da transcrição, sem usar o template nem o fluxo do curso.
- **O que a revisão encontrou:**
  - O formato não seguia o template MADR do `adr-generator`: tinha o campo "Decisores", não tinha "Decision Drivers" nem "Prós e contras das opções".
  - Os ADRs traziam nomes de função, endpoints e tabelas, que o gerador do curso proíbe por serem nível de implementação.
  - Usavam travessões, proibidos pelos prompts do curso.
  - A etapa de Potential ADR foi pulada. A decisão foi formalizada sem antes levantar e pontuar evidências, que é justamente o que o curso aponta como causa de "texto plausível, mas não necessariamente verdadeiro".
  - Uma decisão secundária (snapshot e filtro na inserção da outbox) virou ADR própria.
- **Ação:** os 7 arquivos foram apagados e o trabalho recomeçou seguindo as 3 fases do plugin `adrs-management` (map, identify, generate), com os prompts adaptados em `prompts/`.

## Iteração 1: Potential ADRs e revisão humana

- **O que a IA fez:**
  - Mapeou o código com a transcrição como contexto (`adrs/mapping.md`).
  - Inventariou 45 decisões candidatas, pontuadas com o Step 0 adaptado, e gerou 6 dossiês *must document* e 1 *consider*.
- **Ajuste de prompt necessário:** no plugin original, uma decisão fora do Step 0 soma no máximo 75 pontos e nunca chega a *must document*. Aplicado ao pé da letra numa reunião, isso deixaria sem prioridade até o padrão Outbox. O Step 0 foi adaptado para exigir que a decisão tenha sido fechada explicitamente na reunião.
- **Correções na verificação automática:**
  - Uma citação trazia o travessão que existe no texto original. Foi trocada por `[...]`, porque os prompts do curso proíbem travessões.
  - Uma citação estava com a caixa errada ("mais pra frente" em vez de "Mais pra frente").
- **Decisões humanas na revisão:**
  - O *consider* "Modelo de autorização" não vira ADR e fica como risco no FDD e no PRD.
  - "5 tentativas" são 5 retentativas após o envio inicial, 6 chamadas no total.
  - A data das ADRs é a referência à reunião, que não tem data de calendário.

## Iteração 2: geração das ADRs formais

- **O que a IA fez:** gerou as ADR-001 a ADR-006 com o prompt `prompts/03-adr-generate-adaptado.md`.
- **Correções na verificação automática:**
  - Dois links apontavam para um nome de arquivo inexistente da ADR-004 ("assinatura" em vez de "autenticacao").
  - Uma frase entre aspas era paráfrase, não citação literal. Foi trocada pela fala exata de [09:40] Bruno.
  - A atribuição de "evento de mudança desfeita também não sai" foi corrigida de [09:41] para [09:06] Diego.
  - A referência genérica `prisma/schema.prisma:1` foi trocada por uma linha significativa.
