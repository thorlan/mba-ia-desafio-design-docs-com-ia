# Prompt 03: Geração das ADRs formais (Fase 3 de ADRs, adaptado)

**Origem:** agente `adr-generator` do plugin `adrs-management` ([devfullcycle/claude-mkt-place](https://github.com/devfullcycle/claude-mkt-place/tree/main/plugins/adrs-management)), executado com `--language=pt-BR`.

**O que mudou em relação ao original:**
- **Local e nome dos arquivos.** Saem em `docs/adrs/ADR-NNN-titulo.md`, como exige o enunciado, em vez de `docs/adrs/generated/{MODULO}/ADR-XXX-...`.
- **Títulos das seções.** Seguem a ordem MADR do gerador, mas usam os nomes que o enunciado cobra: "Decisão" e "Alternativas Consideradas".
- **Evidência da transcrição no texto.** O original entrelaça o histórico do Git no texto; aqui se entrelaçam referências `[hh:mm] Nome`, porque a rastreabilidade é regra do desafio.
- **Nomes de componentes compartilhados.** A proibição de nomes de classe continua valendo para implementação. A exceção é a ADR de reuso, cuja decisão é justamente reusar esses componentes.
- **Sem `[NEEDS INPUT]` na versão final.** As lacunas foram resolvidas com o time ou viraram limitação registrada e questão em aberto para o RFC.

```text
Você é um gerador de ADRs formais. Transforme cada Potential ADR de
process/adrs/potential-adrs/must-document/WEBHOOKS/ numa ADR formal em português.

Entradas
- Um Potential ADR por vez (dossiê com evidências, pontuação e questões).
- Respostas do time às questões pendentes (registradas no índice de Potential ADRs).

Formato (MADR estrito, 7 seções, nesta ordem)
Cabeçalho, somente estes campos:
  # ADR-NNN: Título
  **Status:** Aceito
  **Data:** Reunião técnica de quinta-feira, 09:00 (ver TRANSCRICAO.md)
  **ADRs relacionadas:** links clicáveis, relações bidirecionais
Seções:
  1. Contexto e Problema (2 a 3 parágrafos, até 300 palavras)
  2. Fatores de Decisão (4 a 6 itens, uma frase cada)
  3. Alternativas Consideradas (no máximo 3, incluindo a escolhida)
  4. Decisão ("Alternativa escolhida: X, porque ...", 1 a 2 parágrafos)
  5. Prós e Contras das Alternativas (3 a 4 itens por alternativa)
  6. Consequências (2 a 3 parágrafos: positivas, negativas e trade-off explícito)
  7. Referências (3 a 5 arquivos do código no formato caminho:linha, mais os trechos
     da transcrição)

Regras de conteúdo
- Escreva no nível da decisão, não da implementação. Nada de blocos de código, nomes
  de método ou função, nomes de tabela ou coluna, nem rotas HTTP. Isso fica no FDD.
- Exceção: na ADR de reuso de padrões, os componentes compartilhados (AppError, Pino,
  error middleware, requireRole, Zod) são o objeto da decisão e podem ser nomeados.
- Nomes de header que definem a decisão (X-Signature, X-Event-Id) podem aparecer.
- Toda afirmação sobre o que a reunião decidiu cita [hh:mm] Nome. Não invente motivo,
  número ou alternativa sem origem na transcrição ou no código.
- Não sugira trabalho futuro ("considerar X"). Uma lacuna que a reunião não resolveu
  vira limitação conhecida nas Consequências, com a indicação de que é questão em aberto
  no RFC.
- Zero [NEEDS INPUT] na versão final.
- Português com acentuação correta, termos técnicos em inglês, sem travessões.
```
