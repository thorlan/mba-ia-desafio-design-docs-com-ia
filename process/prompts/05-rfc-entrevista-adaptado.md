# Prompt 05: Geração do RFC (entrevista adaptada)

**Origem:** o curso não tem prompt nem template de RFC. Este prompt foi montado no estilo do "Prompt de Entrevista para Gerar PRD para desenvolvimento de Feature" do curso (Notion da Full Cycle), com a mesma estrutura: Objetivo, Papel, Princípios de Entrevista, Regras para coleta, Processo de Entrevista, Perguntas Guia e Esqueleto de saída. As seções do esqueleto são as que o requisito 2 do enunciado exige.

**O que mudou em relação ao prompt de PRD do curso:**
- **Quem responde à entrevista.** No original, o usuário responde a todas as perguntas. Aqui as respostas vêm da transcrição, das ADRs já aprovadas e do índice de Potential ADRs. O usuário só é consultado quando essas fontes não respondem, uma pergunta por vez.
- **Altura do documento.** O PRD pede requisitos, métricas e critérios de aceitação. O RFC fica no nível de arquitetura: proposta, alternativas descartadas e questões em aberto. Contratos, campos, tabelas e códigos de erro ficam para o FDD, como pede o enunciado.
- **Duas origens de questão em aberto.** O enunciado exige pelo menos 2 pontos levantados na reunião e não decididos. As lacunas que a análise das ADRs encontrou (13) também entram, mas separadas, para não se passarem por pontos da reunião.
- **Sem JSON de saída.** O prompt de PRD oferece exportar em JSON. O RFC é entregue só em Markdown.
- **Probabilidade de risco omitida.** A reunião não estimou probabilidades, e o prompt proíbe inventar. Cada risco traz impacto e mitigação com fonte.

```text
Pense profundamente (ultrathink) antes de escrever.

Objetivo
Gerar o RFC da feature "Sistema de Webhooks de Notificação de Pedidos": uma proposta
técnica submetida à equipe para revisão, concisa (2 a 4 páginas), no nível de arquitetura.
O RFC deve responder:
- O que propomos e por quê.
- Quais alternativas foram colocadas na mesa e por que foram descartadas.
- O que ainda está em aberto.
- Qual o impacto no sistema existente e quais os riscos.
O RFC final deve ser renderizado exatamente no formato de "Esqueleto de RFC", em português,
no arquivo docs/RFC.md.

Papel
Você é um assistente focado em RFCs de features de software. Seu papel é:
- Conduzir a entrevista abaixo respondendo cada pergunta a partir das fontes.
- Levar ao usuário só as perguntas que as fontes não respondem, uma por vez, com 2 ou 3
  opções plausíveis.
- Consolidar tudo num documento pronto para revisão.

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

Processo de Entrevista
1. Metadados. Quem é o autor, qual o status e quem revisa.
2. Contexto e problema. Qual a dor, para quem e com que prazo.
3. Proposta técnica. Quais componentes existem, como um evento vai da mudança de status
   até o cliente e que garantias o cliente recebe.
4. Alternativas. Quais alternativas foram discutidas e por que cada uma perdeu.
5. Questões em aberto. O que a reunião adiou ou deixou sem decisão, e o que a análise
   encontrou sem resposta.
6. Impacto e riscos. O que muda no sistema existente, o que pode dar errado e como mitigar.
7. Esforço e dependências. Quanto tempo e do que a entrega depende.
8. Decisões relacionadas. Quais ADRs sustentam a proposta.
Em cada etapa:
- Responda com as fontes, citando.
- Resuma para si o que encontrou.
- Pergunte ao usuário só o que faltar, antes de seguir.

Perguntas Guia
Use como base. Responda pelas fontes; leve ao usuário só o que elas não respondem.
Metadados
- Quem assina o RFC como autor?
- Qual o status do documento (em revisão, aprovado)?
- Qual o papel de cada revisor na reunião?
Contexto e problema
- Quem pediu a feature e por quê?
- Como os clientes resolvem isso hoje e qual o custo?
- Qual a meta de latência e qual o prazo pedido?
Proposta técnica
- Onde nasce o evento e como se garante que ele não se perde nem sai indevidamente?
- Quem entrega o evento e com que frequência?
- O que acontece quando o cliente está fora do ar?
- Como o cliente verifica a origem e reconhece repetições?
- Como a feature se encaixa nos padrões do projeto?
Alternativas
- Quais alternativas foram propostas na reunião e rejeitadas?
- Qual o trade-off que motivou cada descarte, nas palavras de quem falou?
Questões em aberto
- O que a reunião marcou como "observar", "futuro" ou "próxima fase"?
- Quais lacunas as ADRs registraram como questão para o RFC?
Impacto e riscos
- Quais partes do sistema existente mudam?
- Quais riscos a reunião ou as ADRs apontaram, e com que mitigação?
Esforço e dependências
- Qual a estimativa e o que ela inclui?
- De quem ou do que a entrega depende?

Esqueleto de RFC (modelo de saída)
# RFC: Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
| --- | --- |
| Autor | ... |
| Status | ... |
| Data | Reunião técnica de quinta-feira, 09:00 (ver TRANSCRICAO.md) |
| Revisores | Nome (papel), ... |

## 1. Resumo (TL;DR)
Até 5 linhas: problema, proposta e o que se pede aos revisores.

## 2. Contexto e problema
2 a 3 parágrafos, citados.

## 3. Proposta técnica
### 3.1 Visão geral
Um parágrafo e um diagrama Mermaid (flowchart) com os componentes e o caminho do evento.
### 3.2 Componentes e fluxo
Lista curta: cada componente, seu papel e a ADR que o decide.
### 3.3 Garantias e limites
O que o cliente pode esperar (latência, entrega, ordem, autenticidade) e o que não é
garantido.

## 4. Alternativas consideradas
| Alternativa | Levantada em | Trade-off que levou ao descarte | Análise |
Uma linha por alternativa, com link para a ADR na coluna "Análise".

## 5. Questões em aberto
### 5.1 Levantadas na reunião
| Questão | Origem | Situação combinada |
### 5.2 Identificadas na análise das ADRs
| Questão | ADR de origem |
Uma frase explicando que estas não foram discutidas na reunião.

## 6. Impacto e riscos
### 6.1 Impacto no sistema existente
### 6.2 Riscos
| Risco | Impacto | Mitigação | Fonte |
### 6.3 Esforço e dependências

## 7. Decisões relacionadas
Lista com link para cada ADR e uma frase do que ela decide.

Checagens antes de entregar
- Todas as seções do esqueleto estão presentes, nesta ordem.
- Pelo menos 2 alternativas descartadas na reunião, cada uma com trade-off citado.
- Pelo menos 2 questões em 5.1, cada uma com a fala que a adiou.
- Pelo menos 2 ADRs linkadas (o esperado é as 6), com links que resolvem.
- Nenhum contrato, rota, tabela, campo ou código de erro (isso é FDD).
- Nada contradiz as ADRs.
- Todo [hh:mm] Nome existe na transcrição, e toda citação entre aspas é literal.
- Tamanho entre 2 e 4 páginas (cerca de 900 a 1.800 palavras, sem contar tabelas).
- Sem travessões.
```
