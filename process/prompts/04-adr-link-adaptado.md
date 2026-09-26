# Prompt 04: Relações entre as ADRs (Fase 4 de ADRs, adaptado)

**Origem:** agente `adr-linker` do plugin `adrs-management` ([devfullcycle/claude-mkt-place](https://github.com/devfullcycle/claude-mkt-place/tree/main/plugins/adrs-management)).

**Por que só agora:** a Fase 4 não foi executada na primeira versão do PR. A revisão feita por outro modelo apontou a falta dela e relações ausentes entre as ADRs (ver `ITERACOES.md`, Iteração 3). Ela foi executada depois, sobre as seis ADRs já corrigidas.

**O que mudou em relação ao original:**
- **Rótulos em português.** "Depends on", "Used by" e "Related to" viram "Depende de", "Usada por" e "Relacionada a".
- **Sem histórico do Git.** O original usa a data dos commits para detectar supersessão. Aqui todas as decisões são da mesma reunião, então não há supersessão nem emenda, e a ordem temporal é a da numeração.
- **Um único módulo.** Todas as ADRs são do módulo WEBHOOKS e ficam na mesma pasta, então os links usam `./arquivo.md`.
- **Relatórios em `process/adrs/reports/`**, em vez de `docs/adrs/reports/`, para manter `docs/adrs/` só com os arquivos ADR.

```text
Você é o linker de ADRs. Leia todas as ADRs de docs/adrs/ e detecte as relações entre elas.

Tipos de relação, nesta prioridade
1. Depende de: a ADR B cita a decisão da ADR A no Contexto ou na Decisão e não funciona sem ela.
   O inverso, na ADR A, é "Usada por".
2. Relacionada a: as duas tratam de aspectos diferentes do mesmo problema, sem dependência.
   Deve ser recíproca.

Regras
- ADR fundacional (reuso de padrões, framework, utilitários) não é alvo de "Depende de",
  porque todo o resto depende dela de forma transitiva. Pode ser "Relacionada a".
- No máximo 3 links de "Depende de" e 3 de "Relacionada a" por ADR. Se passar, fique com os
  de evidência mais forte e registre no relatório os que foram descartados.
- Toda relação precisa de evidência no texto das duas ADRs. Precisão antes de cobertura.
- Não mude o conteúdo das 7 seções. Só o cabeçalho.

Formato do cabeçalho (substitui "ADRs relacionadas")
  **Status:** ...
  **Data:** ...
  **Depende de:** [ADR-NNN: Título completo](./arquivo.md)
  **Usada por:**
  - [ADR-NNN: Título completo](./arquivo.md)
  - [ADR-NNN: Título completo](./arquivo.md)

  **Relacionada a:** [ADR-NNN: Título completo](./arquivo.md)
Um link: na mesma linha. Dois ou mais: lista em várias linhas. Nunca separado por vírgula.
Linha em branco antes do primeiro "##".

Saída
- Cabeçalhos atualizados.
- process/adrs/reports/adr-link-report.md: relações detectadas, evidência e descartes.
- process/adrs/reports/adr-link-validation.md: links conferidos, quebrados e pares sem
  reciprocidade.
```
