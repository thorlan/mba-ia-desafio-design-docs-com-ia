# Processo de produção

Esta pasta guarda os artefatos intermediários usados para produzir a documentação em `docs/`. Ela não faz parte do pacote de documentos da feature: serve como evidência do workflow descrito no `README.md` da raiz.

| Caminho | Conteúdo |
| --- | --- |
| `prompts/` | Prompts usados em cada etapa, adaptados dos prompts e plugins do curso |
| `adrs/mapping.md` | Fase 1 do fluxo de ADRs: mapeamento do código com a transcrição como contexto |
| `adrs/potential-adrs-index.md` | Fase 2: inventário de todas as decisões candidatas, com pontuação e motivo de descarte |
| `adrs/potential-adrs/` | Fase 2: dossiês (Potential ADRs) que alimentam a geração das ADRs formais; os já formalizados ficam em `done/` |
| `adrs/reports/` | Fase 4: relatório das relações entre as ADRs e validação dos links |
| `adrs/README.md` | Índice das ADRs, fora de `docs/adrs/` para que aquela pasta tenha só os arquivos ADR |
| `ITERACOES.md` | Registro das iterações: o que a IA gerou, o que foi corrigido e por quê |
