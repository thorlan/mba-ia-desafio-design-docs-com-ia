# Design Docs com IA: Sistema de Webhooks de Notificação de Pedidos

Entrega do desafio "Design Docs com IA" do MBA em IA da Full Cycle. O enunciado original está no repositório base: [devfullcycle/mba-ia-desafio-design-docs-com-ia](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia).

## Sobre o desafio

O ponto de partida era uma reunião técnica de 55 minutos, transcrita em [`TRANSCRICAO.md`](TRANSCRICAO.md), em que um time decidiu como adicionar webhooks de notificação de pedidos a um Order Management System existente em Node.js, TypeScript, Express, Prisma e MySQL. A tarefa era transformar essa conversa, mais o código da aplicação, num pacote de design docs: PRD, RFC, FDD, de 5 a 8 ADRs e um Tracker de rastreabilidade, cada um na sua altura, sem repetir conteúdo entre eles.

A regra que mais pesou é a de não inventar nada: toda afirmação precisa ser rastreável a uma fala da reunião ou a um arquivo do código. Tratei isso como o centro do trabalho. A IA escreveu os documentos, e o meu papel foi conduzir: escolher os templates, decidir o que a reunião deixou ambíguo, revisar cada parte antes da próxima e cortar tudo o que não tinha origem.

## Ferramentas de IA utilizadas

- **Claude Code com o modelo Opus 5.5:** leitura do repositório e da transcrição, adaptação dos prompts do curso, geração de todos os documentos, scripts de validação, commits e pull requests.
- **Subagente do Claude Code com o modelo Fable 5.1:** revisão independente das ADRs contra o template do curso. Usei um modelo diferente do que escreveu, para ter um olhar que não compartilhasse os mesmos vieses. A revisão está na Iteração 3.
- **Plugin `adrs-management` e prompts do curso:** não são ferramentas de IA à parte, mas foram a base de todos os prompts. Os agentes do plugin (`adr-analyzer`, `adr-generator`, `adr-linker`) e os prompts de entrevista de PRD e de FDD do Notion do curso foram lidos pelo Claude Code e adaptados para responder a partir da transcrição.

## Workflow adotado

**Ordem de produção.** Segui a ordem sugerida no enunciado: ADRs, RFC, FDD, PRD, Tracker e README. As decisões formam o esqueleto; o RFC consolida a proposta sobre elas; o FDD detalha a implementação; o PRD, por último, consolida em nível de produto.

**Um documento por pull request.** Cada documento teve uma branch `release/<documento>` e um PR no meu fork, e eu revisava antes de passar ao próximo:

| PR | Conteúdo |
| --- | --- |
| #1 | ADRs: mapeamento, 45 candidatos, 7 Potential ADRs, ADR-001 a ADR-006, revisão por outro modelo e fase de link |
| #2 | RFC: prompt e documento |
| #3 | FDD: prompt e documento |
| #4 | PRD: prompt e documento |
| #5 | Regra "se não existe na reunião, remove" aplicada ao FDD, às ADRs e ao RFC |
| #6 | Tracker: prompt e documento, com IDs acrescentados ao RFC, ao PRD e ao FDD |

**Sem template, não escreve.** A primeira tentativa, sem template, foi descartada (Iteração 0). Depois disso, todo documento partiu de um template do curso. Quando o curso não tinha um (RFC e Tracker), o prompt foi montado no mesmo estilo e aprovado por mim num PR antes de gerar o documento.

**Entrevista respondida pela transcrição.** Os prompts do curso são entrevistas com o usuário. Adaptei-os para que as respostas viessem da transcrição, do código e dos documentos já aprovados. Só as lacunas vinham para mim, uma pergunta por vez. Foram poucas: a interpretação das "5 tentativas", o tratamento do modelo de autorização, a data dos documentos, a autoria do RFC e o responsável do FDD e do PRD.

**Validação por script antes de cada PR.** Scripts em Python conferiam:
- se todo `[hh:mm] Nome` existe na transcrição;
- se toda citação entre aspas é literal;
- se todo link resolve e todo arquivo e linha de código citados existem;
- se não há travessões;
- se os checklists do enunciado de cada documento são atendidos.

**Registro do processo.** Tudo o que não é entregável ficou em [`process/`](process/): os prompts adaptados, o mapeamento, os Potential ADRs, o relatório de links entre ADRs e o [`ITERACOES.md`](process/ITERACOES.md), com cada correção e cada decisão humana.

## Prompts customizados

Os oito prompts estão em [`process/prompts/`](process/prompts/), cada um com a origem e o que mudou em relação ao original. Três deles abaixo.

**Prompt 01: mapeamento do código, adaptado do agente `adr-analyzer` do plugin do curso.** O plugin foi feito para reconstruir decisões de sistemas legados a partir do Git. Aqui as decisões ainda não existiam no código, só na reunião, então a transcrição entrou como contexto e o mapeamento passou a responder onde cada decisão vai tocar o código existente.

```text
Você é um analista de arquitetura de software especializado em ADRs. Execute a Fase 1
(mapeamento do codebase) do agente adr-analyzer, com as adaptações abaixo.

Entradas
- Código: o repositório inteiro (src/, prisma/, tests/, package.json, docker-compose.yml,
  tsconfig*.json, .env.example).
- Contexto (equivalente a --context-dir): TRANSCRICAO.md, a reunião técnica que decidiu a
  feature "Sistema de Webhooks de Notificação de Pedidos".

Adaptações
1. O objetivo não é reconstruir decisões antigas. É localizar onde as decisões da reunião
   vão se integrar ao código existente.
2. Inclua um módulo planejado WEBHOOKS, marcado "(a criar)", descrito apenas com o que a
   transcrição afirma. Toda afirmação sobre ele cita [hh:mm] Nome.
3. Na seção de notas de contexto, liste as discrepâncias entre o que a reunião assume e o
   que o código mostra de fato.
4. Toda afirmação sobre o código cita um caminho real (e linha quando ajudar). Nunca cite
   arquivo inexistente, exceto os marcados "(a criar)".
5. Não invente tecnologia, métrica ou componente que não esteja no código ou na transcrição.
6. Português com acentuação correta. Nomes de tecnologia em inglês. Não use travessões.

Saída
process/adrs/mapping.md, seguindo a estrutura do plugin com títulos traduzidos:
Visão geral do projeto, Stack tecnológica, Notas de contexto, Módulos do sistema (com índice
e IDs), Aspectos transversais.
```

**Prompt 05: geração do RFC.** O curso não tem template de RFC. Montei o prompt no estilo do prompt de entrevista de PRD do curso, com as seções exigidas pelo enunciado e duas origens separadas de questão em aberto. Trecho com as fontes e as regras de coleta:

```text
Fontes (nesta ordem de autoridade)
1. docs/adrs/ADR-001 a ADR-006: decisões fechadas e aprovadas. O RFC não as contradiz nem as
   repete por extenso: resume e linka.
2. TRANSCRICAO.md: a reunião. Toda afirmação sobre o que foi dito cita [hh:mm] Nome.
3. process/adrs/potential-adrs-index.md: inventário de 45 candidatos com a classificação
   (ALT, ABERTO, ESCOPO) e a tabela "Questões levadas ao RFC".
4. process/adrs/mapping.md e o código: só para o contexto do sistema existente. Afirmação
   sobre o código cita caminho real.

Princípios de Entrevista
- Uma pergunta por vez ao usuário, e só quando as fontes não respondem.
- Ao final de cada etapa, registre para si um resumo de 3 a 6 linhas do que as fontes
  responderam. Se houver inconsistência entre fontes, pare e pergunte ao usuário.
- O que for dedução sua, e não fato das fontes, fica marcado como hipótese ou vira
  questão em aberto.
Importante:
- Não faça perguntas duplas.
- Não use travessões.
- Não invente nomes, números, prazos, probabilidades ou alternativas sem origem nas fontes.

Regras para coleta de informações
Você deve garantir que capturou:
- Metadados: autor, status, data (referência à reunião, sem data de calendário) e
  revisores. Os revisores são os participantes da reunião, com o papel de cada um.
- Resumo executivo em até 5 linhas.
- Contexto e problema com a dor de negócio e o prazo, citados.
- Proposta técnica em nível de arquitetura: componentes, fluxo de um evento de ponta a
  ponta e garantias oferecidas, com link para a ADR de cada decisão. Sem contratos, rotas,
  tabelas, campos ou códigos de erro (isso é FDD).
- Alternativas consideradas: pelo menos 2 alternativas reais, discutidas e descartadas na
  reunião (classificação ALT no índice), cada uma com o trade-off que levou ao descarte,
  citado, e o link da ADR em que ela é analisada.
- Questões em aberto, em dois grupos:
  a) Levantadas na reunião e adiadas ou não decididas (classificação ABERTO ou ESCOPO com
     adiamento explícito). Pelo menos 2. Cada uma com a fala que a adiou.
  b) Identificadas na análise das ADRs (tabela "Questões levadas ao RFC" do índice), com a
     ADR de origem. Deixe claro que não foram levantadas na reunião.
- Impacto no sistema existente: onde a feature toca o código atual, em nível de módulo.
- Riscos com impacto e mitigação, cada um com fonte. Os riscos de autorização do índice
  (decisão "consider" que não virou ADR) entram aqui de forma resumida.
- Esforço e dependências citados da reunião (estimativa, revisão de segurança).
- Decisões relacionadas: links para as 6 ADRs.
Tudo isso precisa aparecer no RFC final.
```

**Prompt 07: geração do PRD, com a regra de não inventar.** Adaptado do prompt de entrevista de PRD do curso. Depois da minha revisão da primeira versão do PRD, a seção "Defaults Inteligentes" do curso saiu e a regra "se não existe na reunião, remove" entrou. Trecho com os princípios e as regras de coleta:

```text
Princípios de Entrevista
- Uma pergunta por vez ao usuário, e só quando as fontes não respondem.
- Ao final de cada etapa, registre para si um resumo de 3 a 6 linhas do que as fontes
  responderam. Se houver inconsistência entre fontes, pare e pergunte ao usuário.
- O que as fontes não dizem não entra. Um campo obrigatório sem resposta recebe
  "Não definido na reunião".
Importante:
- Não faça perguntas duplas.
- Não use travessões.
- Não invente detalhes que as fontes não deram, nem como hipótese.

Regras para coleta de informações
Você deve garantir que capturou:
- Objetivos claros com métrica e meta alvo. Pelo menos 1 com meta quantitativa.
- O que está dentro do escopo e o que está fora. O "Fora de escopo" lista pelo menos 2
  itens descartados ou adiados na reunião, cada um com a fala que o descartou ou adiou.
- Requisitos funcionais com fluxo principal, variações, erros previstos e prioridade.
  Pelo menos 8, todos discutidos na reunião, cada um com a fala de origem.
- Requisitos não funcionais com metas numéricas ou normas claras, com fonte.
- Arquitetura, componentes, integrações e decisões com justificativa e trade-off, no nível
  de produto, com link para a ADR de cada decisão.
- Dependências reais (técnicas, organizacionais, externas), com quem entrega o quê.
- Riscos com probabilidade, impacto, mitigação e plano de contingência. A probabilidade
  traz a fala da reunião que a sustenta; sem essa fala, o risco não entra. Pelo menos 2
  riscos.
  Os riscos de autorização (decisão "consider" que não virou ADR) entram aqui.
- Checklist objetivo de critérios de aceitação, no nível de produto, sem repetir os
  critérios técnicos do FDD (linkar).
- Estratégia mínima de testes e validação.
- Onde essa feature será implantada (sistema existente).
- IDs: PRD-OBJ-NN, PRD-FR-NN, PRD-NFR-NN, PRD-RISCO-NN, PRD-CA-NN.
Tudo isso precisa aparecer no PRD final.
```

## Iterações e ajustes

O registro completo está em [`process/ITERACOES.md`](process/ITERACOES.md). Foram 9 iterações, numeradas de 0 a 8, depois da exploração inicial. Os momentos em que a IA errou ou ficou superficial e eu tive que corrigir:

1. **ADRs sem template (descartadas).** Logo depois de explorar o repositório, a IA escreveu 7 ADRs direto da transcrição. Elas tinham campos fora do MADR, nomes de função e de tabela proibidos pelo gerador do curso e travessões. A etapa de Potential ADR tinha sido pulada. Mandei parar, passei os templates do curso, e os 7 arquivos foram apagados.
2. **Prompt do curso que não servia para uma reunião.** No plugin de ADR, uma decisão fora do "Step 0" nunca passa de 75 pontos e nunca vira *must document*. Aplicado à reunião, isso deixaria sem prioridade até o padrão Outbox. O Step 0 foi adaptado para exigir que a decisão tenha sido fechada explicitamente na reunião.
3. **Revisão por outro modelo.** Pedi que as ADRs fossem revisadas pelo Fable 5.1. Ele achou 7 problemas de conteúdo que eu confirmei no código e na transcrição antes de corrigir. Entre eles:
   - um erro factual sobre de quais classes derivam os erros de domínio;
   - uma contradição entre ADRs sobre o que a outbox guarda;
   - uma frase que atribuía ao time da reunião uma decisão que foi minha: a leitura de "5 tentativas" como 6 chamadas.

   A revisão também mostrou que a fase de link do plugin não tinha sido feita, e ela foi executada.
4. **Erro que sobreviveu à revisão.** Ao escrever o FDD, a IA percebeu que a correção anterior da ADR-006 estava incompleta: a classe de requisição inválida também aceita código próprio. A ADR foi corrigida no PR do FDD.
5. **Hipóteses no lugar de fatos.** O FDD saiu com 11 hipóteses marcadas, e a primeira versão do PRD trazia p95 de 150 ms, 99,9% de disponibilidade, uma meta de clientes, contingências e testes que ninguém tinha dito. Ao revisar o PRD, defini a regra: se não existe na reunião, remove. Fato do código continua valendo, e campo obrigatório sem resposta diz "Não definido na reunião". A regra foi aplicada ao PRD, ao FDD, às ADRs e ao RFC. Saíram as 11 hipóteses, 7 códigos de erro inventados, prós e contras sem fonte e um risco que vinha da análise.
6. **O que escapou da auditoria.** Ao gerar o Tracker, apareceram três trechos inventados no RFC que o script de auditoria não tinha pegado, porque as linhas já tinham outras citações. Um exemplo é "Seria reativa", sobre Redis Streams. A primeira versão do Tracker também tinha 5 erros de extração, como a fonte de riscos vinda da mitigação e não da coluna de fonte. Os dois foram corrigidos antes do merge.

## Como navegar a entrega

Ordem sugerida de leitura:

1. [`docs/PRD.md`](docs/PRD.md): o problema, o público, o escopo e o que não entra, em nível de produto.
2. [`docs/RFC.md`](docs/RFC.md): a proposta técnica, as alternativas descartadas na reunião e as questões em aberto.
3. [`docs/adrs/`](docs/adrs/): as seis decisões, uma por arquivo, de [ADR-001](docs/adrs/ADR-001-outbox-transacional-no-mysql.md) a [ADR-006](docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md).
4. [`docs/FDD.md`](docs/FDD.md): fluxos, contratos, erros e a integração arquivo por arquivo com o código existente.
5. [`docs/TRACKER.md`](docs/TRACKER.md): a origem de cada item dos documentos.

Para ver como o pacote foi feito:

- [`process/prompts/`](process/prompts/): os oito prompts adaptados.
- [`process/adrs/`](process/adrs/): mapeamento do código, inventário de candidatos, Potential ADRs e relatório de links.
- [`process/ITERACOES.md`](process/ITERACOES.md): cada iteração, correção e decisão humana.

O código da aplicação (`src/`, `prisma/`, `tests/`) e as configurações não foram alterados.
