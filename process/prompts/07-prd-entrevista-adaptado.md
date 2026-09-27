# Prompt 07: Geração do PRD (entrevista adaptada)

**Origem:** "Prompt de Entrevista para Gerar PRD para desenvolvimento de Feature" do curso (Notion da Full Cycle). A estrutura é mantida: Objetivo, Papel, Princípios de Entrevista, Regras para coleta, Processo de Entrevista em 12 etapas, Perguntas Guia, Checagens de Consistência, Defaults Inteligentes, Estilo e Esqueleto de saída.

**O que mudou em relação ao original:**
- **Quem responde à entrevista.** No original, o usuário responde a tudo. Aqui o PRD é o último dos grandes documentos, e o enunciado o descreve como "praticamente uma consolidação". As respostas vêm do RFC, do FDD, das ADRs, da transcrição e do código. O usuário só é consultado quando essas fontes não respondem, uma pergunta por vez.
- **Probabilidade dos riscos mantida.** No RFC e no FDD ela foi omitida, porque a reunião não estimou probabilidades. No PRD o enunciado a exige. Cada probabilidade traz a evidência que a sustenta (uma fala ou um fato do código) ou fica marcada como hipótese, como o próprio prompt do curso manda fazer com os defaults.
- **Defaults Inteligentes só como último recurso.** Os números da reunião vêm primeiro (10 s, 2 s, cerca de 15 h, 64 KB). Os defaults do curso (p95 de 150 ms, 99,9% de disponibilidade) só entram onde a reunião não deu número, marcados como hipótese.
- **Rastreabilidade para o Tracker.** Cada item cita `[hh:mm] Nome` ou `caminho:linha`. Requisitos ganham IDs no formato que o enunciado usa como exemplo (`PRD-FR-01`, `PRD-NFR-01`), e objetivos, riscos e critérios também (`PRD-OBJ-01`, `PRD-RISCO-01`, `PRD-CA-01`).
- **Não repetir o RFC e o FDD.** Decisões e arquitetura ficam no nível de produto, com link para as ADRs, o RFC e o FDD.
- **Sem JSON de saída e sem mensagem inicial**, porque a entrevista não é conduzida com o usuário.
- **Travessões removidos** do texto das regras.

```text
Pense profundamente (ultrathink) antes de escrever.

Objetivo
Gerar o PRD (Product Requirements Document) da feature "Sistema de Webhooks de
Notificação de Pedidos", claro, completo e acionável. O PRD final deve explicar:
- Por que essa feature existe.
- O que ela precisa fazer.
- Como vamos saber que está pronto.
- Em qual sistema ela vai rodar.
O PRD final deve ser renderizado exatamente no formato de "Esqueleto de PRD", em
português, no arquivo docs/PRD.md.

Papel
Você é um assistente focado em PRDs de features de software. Seu papel é:
- Conduzir a entrevista abaixo respondendo cada pergunta a partir das fontes.
- Levar ao usuário só o que as fontes não respondem, uma pergunta por vez, com 2 ou 3
  opções plausíveis.
- Consolidar tudo num documento final pronto para execução.

Fontes (nesta ordem de autoridade)
1. docs/adrs/: decisões fechadas.
2. docs/RFC.md e docs/FDD.md: proposta, garantias, questões em aberto, contratos,
   hipóteses (H1 a H11 do FDD), critérios de aceite técnicos e riscos.
3. TRANSCRICAO.md: a reunião. Cite [hh:mm] Nome.
4. process/adrs/potential-adrs-index.md: inventário com os destinos "PRD" (requisitos
   funcionais, requisitos não funcionais, fora de escopo, dependências).
5. process/adrs/mapping.md e o código, para o contexto do sistema existente. Cite
   caminho:linha.

Princípios de Entrevista
- Uma pergunta por vez ao usuário, e só quando as fontes não respondem.
- Ao final de cada etapa, registre para si um resumo de 3 a 6 linhas do que as fontes
  responderam. Se houver inconsistência entre fontes, pare e pergunte ao usuário.
- Se algo estiver em dúvida, marque como hipótese.
Importante:
- Não faça perguntas duplas.
- Não use travessões.
- Não invente detalhes que as fontes não deram, a menos que ofereça como hipótese marcada.

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
  traz a evidência que a sustenta, ou fica marcada como hipótese. Pelo menos 2 riscos.
  Os riscos de autorização (decisão "consider" que não virou ADR) entram aqui.
- Checklist objetivo de critérios de aceitação, no nível de produto, sem repetir os
  critérios técnicos do FDD (linkar).
- Estratégia mínima de testes e validação.
- Onde essa feature será implantada (sistema existente).
- IDs: PRD-OBJ-NN, PRD-FR-NN, PRD-NFR-NN, PRD-RISCO-NN, PRD-CA-NN.
Tudo isso precisa aparecer no PRD final.

Processo de Entrevista
1. Contexto e visão geral: cenário, público-alvo, onde a feature será implantada e
   objetivo de negócio.
2. Problema e oportunidade: a dor prática, com os números que a reunião deu.
3. Objetivos e métricas de sucesso: objetivo, métrica e meta alvo.
4. Escopo: o que precisa existir e o que fica fora.
5. Requisitos funcionais: nome, descrição, fluxo principal, fluxos alternativos e
   exceções, erros previstos e prioridade.
6. Requisitos não funcionais: performance, disponibilidade, segurança, observabilidade,
   confiabilidade, compatibilidade, compliance, acessibilidade.
7. Arquitetura e abordagem: a visão já existe no RFC; capturar.
8. Decisões e trade-offs: as das ADRs, com justificativa e trade-off.
9. Dependências: o que precisa acontecer para a feature funcionar.
10. Riscos e mitigação.
11. Critérios de aceitação.
12. Testes e validação.
Em cada etapa:
- Responda com as fontes, citando.
- Resuma para si o que encontrou.
- Pergunte ao usuário só o que faltar, antes de seguir.

Perguntas Guia
Use como base. Responda pelas fontes; leve ao usuário só o que elas não respondem.
Contexto e visão
- Qual é o produto ou sistema em que essa feature entra?
- Essa feature pertence a um sistema que já existe ou a um novo sistema?
- Quem é o público-alvo?
- Em duas ou três frases, qual é o objetivo de negócio desta feature?
Problema e oportunidade
- O que está acontecendo hoje que torna essa feature necessária?
- Qual exemplo real, com números aproximados, a reunião deu?
- O que já foi tentado e não funcionou?
Objetivos e métricas de sucesso
- Que resultado mensurável se quer alcançar, com qual métrica e qual meta?
Escopo
- O que precisa obrigatoriamente estar pronto nesta entrega?
- O que está explicitamente fora de escopo?
Requisitos funcionais
- Para cada um: nome, o que o sistema tem que fazer, fluxo principal, variações e
  exceções, quando bloquear ou retornar erro, prioridade.
Requisitos não funcionais
- Performance, disponibilidade, segurança e controle de acesso, observabilidade,
  confiabilidade, compliance, acessibilidade, compatibilidade.
Arquitetura e abordagem
- Onde roda, comunicação síncrona ou assíncrona, fila, cache, integrações externas,
  componentes principais, decisões já dadas e seus trade-offs.
Dependências
- O que precisa chegar de outro time ou área? O que técnico precisa estar pronto antes?
Riscos e mitigação
- Quais os principais riscos? Para cada um: probabilidade (com evidência), impacto,
  mitigação e plano de contingência.
Critérios de aceitação
- Frases objetivas que definem quando a feature está pronta. Nada de "funciona bem".
Testes e validação
- Tipos de teste obrigatórios e abordagem de validação.

Checagens de Consistência antes de finalizar
- Cada objetivo tem métrica e meta alvo.
- Todo requisito funcional tem nome, descrição, fluxo principal, prioridade e fonte.
- Pelo menos 8 requisitos funcionais discutidos na reunião.
- Requisitos não funcionais incluem pelo menos performance e disponibilidade, mesmo que
  marcados como hipótese.
- Fora de escopo não contradiz o que está incluso e tem pelo menos 2 itens da reunião.
- A arquitetura proposta suporta os requisitos não funcionais declarados.
- Toda decisão técnica relevante tem justificativa e trade-off.
- Cada dependência está clara e específica.
- Cada risco tem probabilidade, impacto, mitigação e plano de contingência.
- A checklist de critérios de aceitação está objetiva e verificável.
- Os tipos de teste obrigatórios estão definidos.
- Nada contradiz as ADRs, o RFC ou o FDD.
- Todo [hh:mm] Nome existe na transcrição, e toda citação entre aspas é literal.
- Sem travessões.

Defaults Inteligentes
Use só onde as fontes não dão número. Marque explicitamente como hipótese.
- Latência p95 de APIs síncronas menor que 150 ms.
- Disponibilidade alvo de 99,9% para sistemas voltados ao cliente externo.
- Observabilidade mínima: logs estruturados, métricas de erro por endpoint, tracing.
- Segurança mínima: autenticação, autorização por papel, auditoria de alterações sensíveis.
- Atualizações críticas devem ser transacionais.

Estilo
- Português simples e direto.
- No PRD final, seguir exatamente a estrutura de títulos, subtítulos, negrito e listas do
  esqueleto abaixo, acrescentando só a fonte de cada item e os IDs.

Esqueleto de PRD (modelo de saída)
### PRD: [produto] [feature]

Versão: [versao]
Data: [data]
Responsável: [responsavel_prd]

---

### Resumo

[contexto.resumo]

---

### Contexto e problema

Público-alvo
- [público alvo 1]

Cenários de uso chave
- [cenário 1]

Onde essa feature será implantada
- [contexto_implantacao.descricao]

Problemas priorizados
- [problema com impacto e prioridade]

---

### Objetivos e métricas

| Objetivo | Métrica | Meta |
| --- | --- | --- |
| [objetivo] | [métrica] | [meta] |

---

### Escopo

Incluso
- [item incluso]

Fora de escopo
- [item fora]

---

### Requisitos funcionais

#### [id] [nome do requisito]
[descricao do requisito]

**Fluxo principal**
- [passo]

**Fluxos alternativos e exceções**
- [variação ou exceção]

**Erros previstos**
- [erro previsto]

**Prioridade:** [alta|media|baixa]

---

### Requisitos não funcionais

Performance
- [meta]

Disponibilidade
- [meta]

Segurança e autorização
- [regra]

Observabilidade
- [regra]

Confiabilidade e integridade de dados
- [regra]

Compatibilidade e portabilidade
- [regra]

Compliance
- [regra]

Acessibilidade no frontend consumidor
- [regra]

---

### Arquitetura e abordagem

Abordagem
- [abordagem geral]

Componentes
- [componente]

Integrações
- [integração]

### Decisões e trade-offs

#### Decisão: [decisão]
- **Justificativa:** [por que]
- **Trade-off:** [custo ou limitação]

---

### Dependências

#### [tipo da dependência]: [título]
[quem precisa entregar o quê e por quê]

---

### Riscos e mitigação

#### [risco resumido em uma frase]
- **Probabilidade:** [baixa|media|alta]
- **Impacto:** [impacto esperado]
- **Mitigação:**
  - [ação]
- **Plano de contingência:** [plano B]

---

### Critérios de aceitação
Checklist objetivo que define se a feature está pronta.

- [critério]

---

### Testes e validação

Tipos de teste obrigatórios
- [tipo de teste]

Estratégia de validação
- [estratégia]
```
