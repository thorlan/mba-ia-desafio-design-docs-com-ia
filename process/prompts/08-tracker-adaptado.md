# Prompt 08: Geração do Tracker de Rastreabilidade

**Origem:** o curso não tem prompt nem template de Tracker. O próprio enunciado diz que ele "não é um conceito padrão do mercado nem é um documento abordado diretamente no curso". Este prompt segue a estrutura dos prompts do curso usados nos outros documentos: Objetivo, Papel, Fontes, Regras para coleta, Processo, Esqueleto de saída e Checagens. O formato da tabela e as regras de fonte são os do requisito 5 do enunciado.

**Decisões de adaptação:**
- **Não é entrevista.** O Tracker não cria conteúdo, só aponta a origem do que os outros documentos já dizem. Não há perguntas ao usuário, exceto se um item não tiver origem, o que pela regra "se não existe na reunião, remove" não deveria acontecer.
- **Uma linha por item, com a fonte principal.** O formato do enunciado tem uma coluna "Localização". Quando um item cita várias falas, a linha usa a fala que sustenta o item (a que decide ou define), e as demais continuam no documento.
- **IDs estáveis nos documentos.** O PRD e o FDD já têm IDs. O RFC não tem, e o PRD não tem IDs em "Fora de escopo", "Dependências" e "Decisões". Esses itens recebem IDs no próprio documento, no mesmo PR, para que cada linha do Tracker seja encontrável.
- **Lacunas do RFC 5.2.** Elas registram o que a reunião não definiu. A linha aponta para a fala em que o tema aparece na reunião, com o tipo "Questão em aberto".
- **Localização de código sem linha.** O exemplo do enunciado para `CODIGO` é só o caminho (`src/modules/orders/order.service.ts`). A linha do código, quando importa, vai no resumo do conteúdo.

```text
Pense profundamente (ultrathink) antes de escrever.

Objetivo
Gerar docs/TRACKER.md: uma tabela que mapeia cada item registrado nos documentos do pacote
à sua origem na transcrição ou no código. O Tracker é uma referência cruzada contra
alucinação: todo item rastreado precisa ter origem verificável.

Papel
Você é um auditor de rastreabilidade. Não escreve conteúdo novo; lê os documentos e
aponta de onde veio cada item.

Fontes
- Documentos a rastrear: docs/PRD.md, docs/RFC.md, docs/FDD.md e docs/adrs/ADR-*.md.
- Origens válidas: TRANSCRICAO.md e os arquivos reais de src/, prisma/, tests/,
  package.json e docker-compose.yml.

Regras para coleta
- Formato obrigatório, com estas colunas e nesta ordem:
  | ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
- ID: o ID do item no documento (PRD-FR-01, RFC-ALT-02, FDD-CONTRATO-03, ADR-002).
  Se o item não tiver ID no documento, atribua um no padrão DOC-TIPO-NN e acrescente o
  mesmo ID no documento.
- Documento: caminho do arquivo (docs/PRD.md, docs/RFC.md, docs/FDD.md,
  docs/adrs/ADR-002-...md).
- Tipo: Requisito Funcional, Requisito Não Funcional, Objetivo, Escopo, Fora de Escopo,
  Decisão, Alternativa, Trade-off, Restrição, Questão em Aberto, Risco, Dependência,
  Contrato, Erro, Critério de Aceite, Parâmetro, Integração.
- Conteúdo (resumo): uma linha.
- Fonte: somente TRANSCRICAO ou CODIGO.
- Localização: para TRANSCRICAO, [hh:mm] Nome, exatamente como na transcrição; para
  CODIGO, o caminho do arquivo.
- Uma linha por item. Use a fonte que sustenta o item: a fala que decide ou define, ou o
  arquivo que prova o fato.
- Não rastreie o que o documento marca como "Não definido na reunião" nem os títulos de
  seção.
- Não invente origem: se não achar a fala ou o arquivo, pare e pergunte ao usuário.

Processo
1. Liste os itens identificáveis de cada documento: tudo o que tem ID, cada linha de tabela
   de conteúdo (alternativas, questões, riscos, parâmetros, erros) e cada decisão das ADRs.
2. Para cada item, localize a origem no próprio documento (a citação que ele já traz) e
   confira na transcrição ou no código.
3. Preencha a linha.
4. Calcule a cobertura e as proporções exigidas pelo enunciado.

Esqueleto de saída
# Tracker de Rastreabilidade

[Um parágrafo: o que é, como ler, e os números de cobertura.]

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ... | ... | ... | ... | ... | ... |

Agrupe as linhas por documento, na ordem: PRD, RFC, FDD, ADRs.

Checagens antes de entregar
- A tabela tem exatamente as 6 colunas do enunciado.
- Pelo menos 80% dos itens identificáveis dos documentos têm linha.
- Pelo menos 70% das linhas têm Fonte = TRANSCRICAO, com [hh:mm] Nome que existe na
  transcrição.
- Pelo menos 5 linhas têm Fonte = CODIGO, com caminho de arquivo real.
- Todo ID da tabela existe no documento indicado.
- Nenhum ID duplicado.
- Sem travessões.
```
