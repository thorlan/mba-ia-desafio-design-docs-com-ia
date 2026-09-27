# Prompt 06: Geração do FDD (entrevista adaptada)

**Origem:** "Prompt para geração de um FDD" do curso (Notion da Full Cycle). A estrutura é mantida: Objetivo, Papel, Princípios de Entrevista, Regras para coleta, Processo de Entrevista em 10 etapas e Esqueleto de saída com 10 seções.

**O que mudou em relação ao original:**
- **Quem responde à entrevista.** No original, o usuário responde a tudo. Aqui as respostas vêm das ADRs, do RFC, da transcrição, do mapeamento e do código. O usuário só é consultado quando essas fontes não respondem, uma pergunta por vez.
- **Seção 11, "Integração com o sistema existente".** O enunciado a exige e o template do curso não a tem. Ela nomeia pelo menos 4 arquivos reais e diz como cada um é estendido ou reutilizado.
- **Seção 6 dividida.** O enunciado pede "Matriz de erros" e "Estratégias de resiliência" como itens próprios. As duas viram subseções nomeadas da seção 6 do curso.
- **Rastreabilidade.** O Tracker só aceita `TRANSCRICAO` ou `CODIGO` como fonte. Por isso cada item do FDD cita `[hh:mm] Nome` ou `caminho:linha`, e tudo o que não tiver nenhuma das duas origens fica marcado como hipótese numa lista própria da seção 1. Contratos, erros e critérios ganham IDs (`FDD-CONTRATO-01`, `FDD-ERRO-01`, `FDD-CA-01`) para o Tracker.
- **Questões em aberto do RFC.** O FDD não as decide sozinho. Quando um fluxo precisa de um comportamento para ser implementável, adota um default marcado como hipótese e aponta a questão do RFC.
- **Probabilidade de risco omitida**, como no RFC: a reunião não estimou probabilidades. Cada risco traz impacto, mitigação e plano de contingência com fonte.
- **Sem JSON de saída e sem mensagem inicial**, porque a entrevista não é conduzida com o usuário.
- **Travessões removidos do esqueleto.** O original separa status e significado com travessão; aqui o separador é dois-pontos.

```text
Pense profundamente (ultrathink) antes de escrever.

Objetivo
Gerar o FDD (Feature Design Doc) da feature "Sistema de Webhooks de Notificação de Pedidos":
o documento que diz como implementar, acionável o bastante para um desenvolvedor pegar e
começar a codar. O FDD detalha fluxos, contratos públicos, erros, observabilidade, critérios
de aceite técnicos, riscos e a integração com o código existente. Não repete a narrativa de
negócio do PRD nem a justificativa das ADRs: linka.
O FDD final deve ser renderizado exatamente no formato de "Esqueleto de FDD", em português,
no arquivo docs/FDD.md.

Papel
Você é um assistente especializado em FDD. Seu papel é:
- Conduzir a entrevista abaixo respondendo cada pergunta a partir das fontes.
- Levar ao usuário só o que as fontes não respondem, uma pergunta por vez, com 2 ou 3
  opções plausíveis.
- Consolidar tudo num documento técnico que permita implementação sem ambiguidade e
  validação objetiva.

Fontes (nesta ordem de autoridade)
1. docs/adrs/ADR-001 a ADR-006: decisões fechadas. O FDD implementa, não rediscute.
2. docs/RFC.md: proposta, garantias e limites, questões em aberto (5.1 e 5.2) e riscos.
3. TRANSCRICAO.md: detalhes de implementação combinados na reunião (rotas, campos, headers,
   payload, códigos de erro, timeout, limites). Cite [hh:mm] Nome.
4. process/adrs/mapping.md e o código (src/, prisma/, tests/, package.json): padrões a
   seguir e pontos de integração. Cite caminho:linha.

Princípios de Entrevista
- Uma pergunta por vez ao usuário, e só quando as fontes não respondem.
- Ao final de cada etapa, registre para si um resumo de 3 a 6 linhas do que as fontes
  responderam. Se houver inconsistência entre fontes, pare e pergunte ao usuário.
- Não invente detalhes técnicos sem rotular como hipótese.
- Não use travessões.

Regras para coleta de informações
Garanta capturar, no mínimo, as seções do esqueleto. Além disso:
- Rastreabilidade: todo item cita [hh:mm] Nome ou caminho:linha. O que não tiver nenhuma das
  duas origens entra em "Suposições (hipóteses)" na seção 1, com o motivo. Mantenha essa lista
  curta: prefira o que a reunião ou o código já definem.
- IDs: contratos FDD-CONTRATO-NN, erros FDD-ERRO-NN, critérios de aceite FDD-CA-NN, riscos
  FDD-RISCO-NN.
- Parâmetros configuráveis e defaults: intervalo de polling, tamanho do lote, timeout,
  progressão do backoff, teto de payload, carência da rotação. Cada default com fonte.
- Contratos públicos: pelo menos 4 endpoints HTTP da API, cada um com rota completa (o
  prefixo real das rotas está em src/app.ts), método, exemplo de requisição, exemplo de
  resposta, status codes e semântica. Inclua também o contrato de saída (a chamada que o
  worker faz ao cliente: headers, payload, respostas esperadas) e a função interna de
  integração com pedidos.
- Matriz de erros: todo código com prefixo WEBHOOK_, com status HTTP, condição e
  tratamento. Os códigos citados na reunião vêm primeiro; um código novo, exigido por um
  fluxo, fica marcado como hipótese. Registre o conflito entre a validação no schema e o
  código próprio do módulo (RFC 5.2) e o default adotado.
- Observabilidade: métricas, logs e tracing. O projeto não tem biblioteca de métricas nem
  de tracing, e a ADR-006 veta biblioteca nova. Proponha métricas derivadas do banco e dos
  logs estruturados do Pino, e tracing por correlação de identificadores (o identificador
  do evento e o identificador de requisição que o projeto já gera), marcando como hipótese
  o que não vier das fontes. Inclua proteção de dados sensíveis (secret fora dos logs).
- Questões em aberto do RFC que afetam um fluxo: adote um default marcado como hipótese e
  cite o item do RFC. Não apresente o default como decisão do time.
- Riscos: impacto, mitigação (com subitens quando houver mais de uma) e plano de
  contingência, cada um com fonte. Os riscos de autorização (decisão "consider" que não
  virou ADR) entram aqui com detalhe de implementação.
- Seção 11: pelo menos 4 caminhos de arquivo reais, cada um com o que muda ou o que é
  reutilizado e como. Confira que cada caminho e linha existem.

Processo de Entrevista
1. Contexto e motivação técnica. Qual problema técnico a feature resolve, como se encaixa
   no sistema existente, quem são os atores, quais os limites, suposições e restrições.
2. Objetivos técnicos. Quais resultados mensuráveis e quais invariantes precisam valer.
3. Escopo e exclusões. O que entra nesta entrega e o que fica fora.
4. Fluxos detalhados. Criação do evento na outbox, processamento pelo worker, retry, DLQ,
   replay, rotação de secret. Onde há validação, persistência e chamada externa.
   Diagramas de sequência e de estados do evento.
5. Contratos públicos. Endpoints da API, contrato de saída para o cliente e função
   interna, com exemplos e semântica.
6. Erros, exceções e fallback. Matriz de erros, estratégias de resiliência, fallback e
   invariantes.
7. Observabilidade. Métricas, logs, tracing, dashboards e alertas.
8. Dependências e compatibilidade. Versões do que já existe no projeto e impacto em
   interfaces existentes.
9. Critérios de aceite técnicos. Checklist objetivo com metas numéricas, seguindo os
   padrões de teste de tests/.
10. Riscos e mitigação.
11. Integração com o sistema existente. Arquivo por arquivo.
Em cada etapa:
- Responda com as fontes, citando.
- Resuma para si o que encontrou.
- Pergunte ao usuário só o que faltar, antes de seguir.

Esqueleto de FDD (modelo de saída)
# FDD: Sistema de Webhooks de Notificação de Pedidos

Versão: [versão]
Data: Reunião técnica de quinta-feira, 09:00 (ver TRANSCRICAO.md)
Responsável: [responsável técnico]
Documentos relacionados: [RFC](RFC.md), [ADRs](adrs/)

## 1. Contexto e motivação técnica
[problema técnico, encaixe no sistema existente, atores e limites]
**Restrições**
- [restrição com fonte]
**Suposições (hipóteses)**
- [hipótese e motivo]

## 2. Objetivos técnicos
- [objetivo com medida ou invariante, e fonte]

## 3. Escopo e exclusões
**Incluído**
- [item com fonte]
**Excluído**
- [item com fonte]

## 4. Fluxos detalhados e diagramas
**Fluxo principal**
1. [passo]
**Fluxos alternativos e exceções**
- [variação]
**Diagramas**
[Mermaid: sequência do fluxo principal e estados do evento]

## 5. Contratos públicos (assinaturas, endpoints, headers, exemplos)
### FDD-CONTRATO-NN: [nome]
- Tipo: [function|method|endpoint]
- Assinatura/Rota: [rota completa ou assinatura]
- Método: [GET|POST|...]
- Semântica de status/headers:
  - [status ou header]: [significado]
- Fonte: [hh:mm] Nome ou caminho:linha

**Exemplo de requisição**
(bloco json)

**Exemplo de resposta**
(bloco json)

## 6. Erros, exceções e fallback
### 6.1 Matriz de erros previstos
| ID | Código | HTTP | Condição | Tratamento | Fonte |
### 6.2 Estratégias de resiliência
- Timeouts, retries, backoff, e o que não se aplica (circuit breaker) e por quê.
### 6.3 Política de fallback
### 6.4 Invariantes
- [invariante crítico]

## 7. Observabilidade
**Métricas**
- [métrica, como é obtida]
**Logs**
- Formato e campos essenciais, e o que nunca é logado.
**Tracing**
- Spans ou correlação principal e amostragem.
**Dashboards e alertas**
- [painel ou alerta mínimo]

## 8. Dependências e compatibilidade
**Dependências**
- [componente, versão do package.json ou do docker-compose.yml]
**Garantias de compatibilidade**
- [impacto em interfaces existentes]

## 9. Critérios de aceite técnicos
- [ ] FDD-CA-NN: [critério objetivo]

## 10. Riscos e mitigação
### FDD-RISCO-NN: [risco]
- Impacto: [impacto]
- Mitigação:
  - [ação]
- Plano de contingência: [plano B]
- Fonte: [hh:mm] Nome ou caminho:linha

## 11. Integração com o sistema existente
### [caminho do arquivo]
- O que existe hoje: [com linha]
- O que muda ou é reutilizado: [como]

Checagens antes de entregar
- As 11 seções do esqueleto estão presentes, nesta ordem.
- Pelo menos 4 endpoints HTTP, cada um com exemplo de requisição, exemplo de resposta e
  status codes.
- Todos os códigos da matriz de erros usam o prefixo WEBHOOK_.
- A seção 7 cita métricas, logs e tracing.
- A seção 11 cita pelo menos 4 caminhos reais, e cada caminho e linha citados existem.
- Nada contradiz as ADRs nem o RFC. Nenhuma questão em aberto do RFC aparece como decidida.
- Todo [hh:mm] Nome existe na transcrição, e toda citação entre aspas é literal.
- Toda hipótese está na lista da seção 1 ou marcada no ponto em que aparece.
- Os exemplos JSON são válidos e usam os nomes de campo combinados na reunião.
- Sem travessões.
```
