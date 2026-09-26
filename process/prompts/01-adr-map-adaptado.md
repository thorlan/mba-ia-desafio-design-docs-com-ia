# Prompt 01: Mapeamento do código (Fase 1 de ADRs, adaptado)

**Origem:** agente `adr-analyzer`, Fase 1, do plugin `adrs-management` ([devfullcycle/claude-mkt-place](https://github.com/devfullcycle/claude-mkt-place/tree/main/plugins/adrs-management)).

**O que mudou em relação ao original:** o plugin foi feito para reconstruir decisões de sistemas legados a partir do código e do Git. Aqui as decisões ainda não existem no código; elas estão na transcrição da reunião. Por isso a transcrição entra como `--context-dir` e o mapeamento passa a responder "onde cada decisão vai tocar o código existente".

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
