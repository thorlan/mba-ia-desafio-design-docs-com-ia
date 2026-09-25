# Prompt 02: Identificação de Potential ADRs (Fase 2 de ADRs, adaptado)

**Origem:** agente `adr-analyzer`, Fase 2, do plugin `adrs-management` ([devfullcycle/claude-mkt-place](https://github.com/devfullcycle/claude-mkt-place/tree/main/plugins/adrs-management)).

**O que mudou em relação ao original:**
- A fonte das decisões é a transcrição, não o código.
- O "Step 0" (base score automática) passa a exigir que a decisão tenha sido fechada explicitamente na reunião. No plugin original, uma decisão fora do Step 0 soma no máximo 75 pontos, então nunca chega a `must-document`. Sem essa adaptação, nenhuma decisão tirada de uma reunião seria priorizada.
- O Git é substituído pela linha do tempo da reunião (`[hh:mm] Nome`).

```text
Execute a Fase 2 do adr-analyzer (identificação de Potential ADRs) usando
process/adrs/mapping.md e TRANSCRICAO.md.

1. Inventário completo
   Varra a transcrição do início ao fim e liste TODAS as decisões candidatas, inclusive as
   descartadas, adiadas e secundárias. Cada uma com [hh:mm] Nome de onde surgiu e de onde
   foi fechada.

2. Step 0 adaptado (base score)
   O candidato só recebe base score se atender às duas condições:
   a) foi fechado explicitamente como decisão na reunião ("decidido", "decisão",
      "tá decidido", "anotado" como decisão, ou consta no resumo final de [09:48] Larissa);
   b) cai numa categoria estrutural do plugin: infraestrutura, framework/plataforma,
      camada de dados, protocolo ou contrato de API, ou infraestrutura crítica do domínio
      (autenticação e autorização, entrega de notificações).
   Base 75 para as categorias universais (1 a 4 do plugin) e 70 para domínio crítico.
   Candidatos fora do Step 0 começam em 0.

3. Filtros do plugin, sem alteração
   Aplique os Red Flags (1 a 5) aos candidatos fora do Step 0 e a regra dos 3 E's
   (Estrutural, Evidente, Estável) a todos. Parâmetros como intervalo de polling, número de
   tentativas, timeout e limite de payload são Red Flag 3 ou 5: consolidar na decisão maior.

4. Respeite a própria reunião
   Quando um participante classifica algo como "não é decisão arquitetural" (por exemplo
   [09:23] Sofia e [09:24] Larissa), isso vale como descarte com essa justificativa.

5. Pontuação
   Some base + as 3 dimensões do plugin (Escopo e impacto, Custo de mudança, Conhecimento do
   time, 0 a 25 cada), justificando cada nota em uma linha.
   >= 100: must-document. 75 a 99: consider. < 75: descartar (registrar o destino: FDD, PRD,
   fora de escopo ou questão em aberto).

6. Evidências
   - Transcrição: citação curta e literal com [hh:mm] Nome.
   - Código: caminhos reais (com linha) dos pontos que a decisão toca.
   - "Impact Analysis" do plugin vira "Linha do tempo na reunião".

7. Não resolva lacunas
   O que a reunião não respondeu vira item de "Questões para a ADR". Marque
   [NEEDS INPUT: ...] quando a resposta depender de alguém do time.

8. Português, sem travessões.

Saída
- process/adrs/potential-adrs-index.md: inventário completo (inclusive descartados, com o
  motivo e o destino) e as tabelas must-document e consider.
- Um arquivo por candidato com score >= 75 em
  process/adrs/potential-adrs/{must-document|consider}/WEBHOOKS/<titulo-em-kebab-case>.md,
  no template de Potential ADR do plugin, com títulos traduzidos.
```
