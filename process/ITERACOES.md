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

## Iteração 4: RFC

- **Prompt:** o curso não tem template de RFC. O prompt `prompts/05-rfc-entrevista-adaptado.md` segue a estrutura do prompt de entrevista do PRD do curso e foi aprovado pelo usuário no PR antes de gerar o documento.
- **Entrevista:** as respostas vieram das ADRs, da transcrição e do índice de Potential ADRs. A única lacuna levada ao usuário foi a autoria. Ele escolheu Larissa (Tech Lead) como autora, e os outros quatro participantes ficaram como revisores.
- **Questões em aberto separadas por origem:**
  - 5.1 traz os 5 pontos que a reunião adiou ou deixou sem decisão: rate limiting, vários workers, autorização do cadastro, e-mail de falha e arquivamento.
  - 5.2 traz as lacunas encontradas na análise, marcadas como não discutidas na reunião.
- **Lacuna nova:** o mapeamento do código já apontava que a criação do pedido grava o status inicial fora da mudança de status (discrepância 7), mas ela não estava na lista de questões para o RFC. Foi acrescentada ao índice e ao RFC (agora são 14 na seção 5.2).
- **Correções na verificação automática:** três citações tinham caixa ou aspas diferentes do original ("Observar", "Problema do futuro" e aspas simples dentro de uma citação). Foram ajustadas para bater literalmente com a transcrição.

## Iteração 5: FDD

- **Prompt:** `prompts/06-fdd-entrevista-adaptado.md`, adaptado do prompt de FDD do curso (extraído do Notion) e aprovado pelo usuário no PR antes da geração.
- **Entrevista:** as respostas vieram das ADRs, do RFC, da transcrição e do código. A única lacuna levada ao usuário foi o responsável técnico. O usuário perguntou se o enunciado define isso; a busca mostrou que não (só o RFC tem metadados, e sem autor definido). A decisão foi Larissa no FDD, alternando com Diego nos próximos documentos.
- **Hipóteses declaradas:** 11 defaults (H1 a H11) na seção 1, cada um ligado à questão do RFC que ele destrava, para o FDD ser implementável sem apresentar como decisão do time o que a reunião não decidiu.
- **Erro encontrado numa ADR já mergeada:** ao ler as classes de erro para a matriz, a IA viu que a correção I1 da ADR-006 (Iteração 3) dizia que só as classes de conflito e de entidade não processável aceitam código próprio. A de requisição inválida também aceita (`src/shared/errors/http-errors.ts:4`). A ADR-006 foi corrigida neste PR. Esse detalhe importa, porque é por essa classe que `WEBHOOK_INVALID_URL` sai com status 400.
- **Correções na verificação automática:** três referências de linha estavam erradas (paginação em `response.ts`, versão do Prisma e faixa de dependências no `package.json`). A causa foi ler arquivos concatenados, com a numeração contínua entre eles.

## Iteração 6: PRD

- **Prompt:** `prompts/07-prd-entrevista-adaptado.md`, adaptado do prompt de entrevista de PRD do curso e aprovado pelo usuário no PR antes da geração.
- **Entrevista:** as fontes (RFC, FDD, ADRs, transcrição e código) responderam todas as etapas. O responsável já estava decidido (Diego, pelo revezamento da Iteração 5), então nenhuma pergunta foi levada ao usuário.
- **Probabilidade dos riscos:** o enunciado exige probabilidade no PRD, e a reunião não estimou nenhuma. Cada uma veio com a evidência que a sustenta, por exemplo "Nenhum evento nosso vai chegar perto disso" ([09:24] Diego) para o limite de payload, ou ficou marcada como hipótese.
- **Defaults do curso:** usados só onde a reunião não deu número (p95 de 150 ms nas rotas de cadastro e 99,9% de disponibilidade), marcados como hipótese.
- **Títulos:** o PRD segue os títulos exatos do esqueleto do curso (com `###`), porque o prompt original pede isso explicitamente, diferente do RFC e do FDD.
- **Revisão humana: "se não existe na reunião, remove".** O usuário leu a primeira versão do PRD e definiu uma regra que vale daqui em diante: nada de hipóteses. Foram removidos:
  - os defaults do curso (p95 de 150 ms e 99,9% de disponibilidade) e a meta de 3 clientes integrados;
  - um risco que não vinha da reunião (worker parado) e as contingências e mitigações inventadas;
  - erros e fluxos alternativos deduzidos, como "customer inexistente" e "lista vazia";
  - requisitos vindos do FDD e não da reunião (mascaramento da secret, métricas, DLQ para payload grande);
  - os testes unitários e de worker e a homologação com cliente.
  Fatos do código continuaram, porque são verificáveis. Campos obrigatórios do template sem resposta na reunião passaram a dizer "Não definido na reunião". A probabilidade do risco de prazo mudou de "média (hipótese)" para "baixa", com a evidência "Atlas vai gostar" ([09:47] Marcos). O prompt 07 foi atualizado com a regra, e a seção "Defaults Inteligentes" saiu dele.

## Iteração 7: "se não existe na reunião, remove" aplicado a todo o pacote

- **Regra do usuário:** o que não existe na reunião sai do documento, sem hipóteses nem inferências, e sem deixar de cumprir o enunciado. Fatos do código continuam valendo. Campo obrigatório sem resposta na reunião diz "Não definido na reunião".
- **FDD:**
  - Saíram as 11 hipóteses (H1 a H11). No lugar entrou a lista "Não definido na reunião" na seção 1, que aponta para o RFC 5.2 sem escolher por ele.
  - A matriz de erros ficou só com os 3 códigos citados em [09:28] Bruno. Os 7 códigos inventados saíram, e os demais erros previstos aparecem sem código.
  - Saíram também: a rota e o método da rotação de secret (a reunião definiu o endpoint, não a rota), o formato da assinatura, a dupla assinatura na carência, a remoção lógica, a DLQ para payload grande, o tamanho do lote, os campos de tentativas e as métricas, os alertas e os logs inventados.
  - A observabilidade passou a registrar que a reunião não definiu métricas nem tracing, e mostra só os dados definidos na reunião e o que o código já oferece.
  - Os status HTTP de sucesso seguem a convenção dos controllers existentes.
  - Com isso, o FDD continua cumprindo o enunciado: 6 endpoints com rota, request, response e status, e matriz com `WEBHOOK_*`.
- **ADRs:**
  - Saíram prós e contras que ninguém disse, como "uma única secret para gerenciar", "ponto único de falha" e "reduz a superfície de revisão". Também saíram exemplos deduzidos e frases de análise, como "quem conhece um módulo conhece o de webhooks".
  - O que tinha origem ganhou citação. Os trade-offs explícitos, que o enunciado exige, foram reescritos só com falas citadas.
- **RFC:** saiu o risco "worker de instância única para ou fica lento", que vinha da análise e não da reunião.
- **Mantido:** as lacunas ("a reunião não definiu X") continuam nas ADRs e no RFC 5.2, porque são afirmações verdadeiras sobre a reunião, equivalentes ao "Não definido na reunião".

## Iteração 8: Tracker

- **Prompt:** `prompts/08-tracker-adaptado.md`, aprovado pelo usuário no PR antes da geração. O curso não tem template de Tracker; o prompt segue o formato do requisito 5 do enunciado.
- **IDs nos documentos:** com a aprovação do usuário, os itens sem ID receberam um no próprio documento, para cada linha do Tracker ser encontrável. No RFC: componentes, alternativas, questões em aberto e riscos. No PRD: escopo, fora de escopo, dependências e decisões. No FDD: restrições, objetivos, parâmetros e integração.
- **Geração por script:** para cada ID, a fonte é a citação que o próprio documento traz no item (a linha "Fonte:", a coluna "Fonte" ou a primeira citação). O script valida que todo timestamp existe, que todo caminho de código existe, que todo ID está no documento indicado e que não há ID duplicado.
- **Revisão das fontes:** a primeira versão tinha 5 problemas de extração, corrigidos antes do commit:
  - o resumo dos objetivos do PRD vinha da coluna de métrica;
  - resumos de "Fora de escopo" cortados no meio de uma citação;
  - um ")" sobrando nas decisões do PRD;
  - riscos do RFC com a fonte da mitigação, e não da coluna "Fonte";
  - dois riscos do PRD apontando para a evidência da probabilidade, e não para a fala que levanta o risco.
- **Invenções que tinham escapado da Iteração 7:** ao colocar IDs no RFC, a IA encontrou três trechos sem fonte na tabela de alternativas: "Seria reativa" (Redis), "Uma secret só seria mais simples de gerenciar" e "Eliminaria repetições". Foram trocados pelas falas literais. O script de auditoria da Iteração 7 não os pegou porque as linhas já tinham outras citações.
- **Resultado:** 184 linhas, 184 de 184 itens identificados (100%), 172 com `TRANSCRICAO` (93%) e 12 com `CODIGO`.
