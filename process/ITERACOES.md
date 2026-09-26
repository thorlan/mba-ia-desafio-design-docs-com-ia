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
- **Ação:** os 7 arquivos foram apagados e o trabalho recomeçou seguindo as fases do plugin `adrs-management` (map, identify, generate; a fase de link veio na Iteração 3), com os prompts adaptados em `prompts/`.

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

## Iteração 3: revisão das ADRs por outro modelo

- **Pedido:** revisar as ADRs com outro modelo (Fable 5.1, diferente do que as escreveu) e conferir se seguem o template do `adr-generator`.
- **Resultado:** nenhum achado bloqueia o enunciado. Timestamps, citações, linhas de código e links conferem. Foram apontados 7 problemas de conteúdo, todos confirmados no código e na transcrição antes de corrigir:
  - **I1, ADR-006:** afirmava que as classes de domínio estendem `AppError` diretamente. No código elas derivam de `ConflictError` e `UnprocessableEntityError` (`http-errors.ts:45` e `:55`).
  - **I2, ADR-003:** dizia que a outbox guarda só eventos em andamento, o que contradiz a ADR-001 e [09:08] Diego, segundo quem ela também guarda os entregues.
  - **I3, ADR-003:** "confirmada com o time" atribuía ao time da reunião uma decisão do autor (as 6 chamadas). O rótulo "Decisões do time" no índice e "Respostas do time" no prompt 03 induziam o erro e foram trocados.
  - **I4, ADR-004:** atribuía a [09:46] Sofia a proteção da secret em repouso, tema que ela não citou. Virou lacuna e questão para o RFC.
  - **I5, ADR-002:** apresentava como trade-off aceito uma inferência da IA sobre ordem em retentativa. O trade-off agora usa só o que [09:13] Larissa aceitou.
  - **I6, ADR-006:** a terceira alternativa (injetar repositório) respondia a outra pergunta. Foi removida, e a escolha continua registrada na Decisão.
  - **I7, ADR-006:** o conflito entre a validação no schema Zod e `WEBHOOK_INVALID_URL`, registrado na Etapa 1, não estava na lista de questões para o RFC.
- **Efeito:** a lista de questões levadas ao RFC passou de 11 para 13.
- **Ajustes menores, também apontados na revisão:**
  - Decisão com no máximo 2 parágrafos e sempre com o "porque", como pede o template (ADR-001 a ADR-006).
  - Detalhes de implementação removidos: função que recebe a transação (ADR-001), entry point e conexão de ORM (ADR-002), campos da DLQ (ADR-003) e campos da configuração (ADR-004).
  - Prescrições trocadas por constatações neutras: mascaramento de secrets (ADR-004) e erros do worker (ADR-006).
  - Afirmações sem fonte removidas ou citadas: um driver inteiro da ADR-006, a não atomicidade do broker (ADR-001, agora com [09:41] Diego), a disputa de recursos (ADR-002) e a correlação de logs pelo identificador (ADR-005).
  - ADR-003: a terceira alternativa misturava dois eixos. Ficou só "teto de 3 tentativas", proposta de [09:16] Bruno, e a escolha de DLQ em tabela própria em vez de marcar falha na outbox passou para a Decisão.
  - ADR-006: o módulo de autenticação não tem repository, o que agora está dito.
  - Linha "Transcrição" das Referências alinhada ao corpo em todas as ADRs, e `docker-compose.yml` com número de linha.
- **Fluxo do plugin completado:**
  - A Fase 4 (`adr-link`) não tinha sido feita. Foi executada com o prompt `prompts/04-adr-link-adaptado.md`: os cabeçalhos passaram a ter relações tipadas ("Depende de", "Usada por", "Relacionada a"), e o relatório e a validação estão em `adrs/reports/`.
  - Os 6 dossiês formalizados foram arquivados em `adrs/potential-adrs/done/WEBHOOKS/`, com os links do índice e do dossiê *consider* ajustados.
- **Desvios do template declarados** no prompt 03: tamanho abaixo de 100 linhas, linha da transcrição fora do limite de 5 referências, códigos de erro na ADR de reuso e ausência dos níveis `generated/` e `needs-input/`.
- **Estrutura:** o índice `docs/adrs/README.md` foi para `process/adrs/README.md`, porque o critério do enunciado pede que `docs/adrs/` contenha só arquivos no formato `ADR-NNN-*.md`.
