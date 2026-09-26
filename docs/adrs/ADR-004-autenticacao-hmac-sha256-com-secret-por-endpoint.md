# ADR-004: Autenticação HMAC-SHA256 com secret por endpoint e rotação

**Status:** Aceito
**Data:** Reunião técnica de quinta-feira, 09:00 (ver `TRANSCRICAO.md`)
**Relacionada a:**
- [ADR-002: Worker em processo separado com polling](./ADR-002-worker-em-processo-separado-com-polling.md)
- [ADR-005: Entrega at-least-once com X-Event-Id para deduplicação](./ADR-005-entrega-at-least-once-com-x-event-id.md)

## Contexto e Problema

Os webhooks só saem da plataforma para o cliente ([09:02] Marcos, [09:03] Sofia) e levam dados de pedidos a endpoints fora da nossa infraestrutura. O cliente precisa conseguir verificar duas coisas: que a requisição veio de nós e que ninguém alterou o payload no caminho ([09:19] Sofia).

O histórico reforça o cuidado com segredos: um cliente já vazou uma secret num log da própria aplicação ([09:22] Diego). O mecanismo escolhido precisa limitar o estrago de um vazamento e permitir a troca da secret sem interromper as entregas.

## Fatores de Decisão

- Usar um padrão de mercado que qualquer cliente consiga verificar com bibliotecas comuns ([09:20] Sofia).
- Limitar o alcance de um vazamento a um único endpoint ([09:21] Sofia).
- Permitir a troca da secret sem janela de falha para o cliente ([09:21] Sofia).
- Proteger o tráfego com transporte cifrado ([09:23] Sofia).
- Passar pela revisão de segurança antes do deploy ([09:46] Sofia).

## Alternativas Consideradas

1. **HMAC-SHA256 com secret única por endpoint e rotação com carência.**
2. **HMAC com uma secret global da plataforma.**

## Decisão

Alternativa escolhida: **HMAC-SHA256 calculado sobre o corpo da requisição, com secret única por endpoint de webhook e rotação com carência de 24 horas** ([09:22] Sofia), porque é um padrão que qualquer cliente verifica ([09:20] Sofia) e um vazamento compromete um único endpoint ([09:21] Sofia).

A assinatura vai no header `X-Signature` ([09:20] Sofia). A secret é gerada pela plataforma e entregue ao cliente na criação do webhook ([09:31] Marcos). Ela fica guardada junto da configuração do endpoint ([09:21] Bruno, [09:21] Sofia). O cliente pode pedir uma nova secret pela API; a anterior continua válida por 24 horas em paralelo e depois é invalidada ([09:21] Sofia). Só são aceitas URLs com TLS. Essa exigência foi tratada na reunião como validação de entrada, não como decisão arquitetural ([09:23] Sofia).

## Prós e Contras das Alternativas

### HMAC-SHA256 com secret por endpoint e rotação
- Pró: padrão de mercado, com suporte em qualquer linguagem ([09:20] Sofia).
- Pró: um vazamento compromete um único endpoint.
- Pró: a rotação com carência evita janela de falha na troca.
- Contra: exige gerar, guardar, rotacionar e expirar secrets por endpoint.

### HMAC com secret global da plataforma
- Pró: uma única secret para gerenciar.
- Contra: "se vaza uma, vaza tudo" ([09:21] Sofia).
- Contra: trocar a secret afetaria todos os clientes ao mesmo tempo.
- Contra: a plataforma não conseguiria isolar um cliente comprometido.

## Consequências

**Positivas.** Cada cliente verifica a origem e a integridade de cada entrega com uma técnica conhecida. Um vazamento, como o que já aconteceu ([09:22] Diego), afeta um único endpoint e pode ser corrigido por rotação, sem janela de falha.

**Negativas.** Diferente das senhas de usuário, guardadas como hash irreversível, a secret precisa ficar recuperável, porque o worker a usa para assinar cada envio. A reunião não definiu como a secret é protegida em repouso. O logger atual mascara senhas e tokens, mas não secrets. A assinatura cobre só o corpo da requisição, e por isso o timestamp de envio mandado em header ([09:44] Diego) não é autenticado por si só. A reunião também não definiu como a mensagem é assinada durante as 24 horas em que duas secrets são válidas. Os três pontos ficam como questões em aberto no RFC.

**Trade-off explícito:** aceitamos a complexidade de gerenciar secrets recuperáveis por endpoint, com rotação, em troca de limitar o alcance de um vazamento e oferecer ao cliente uma verificação padrão de mercado.

## Referências

- `src/modules/users/user.service.ts:26` (senhas guardadas como hash; padrão que não se aplica à secret)
- `src/shared/logger/index.ts:4` (lista de campos mascarados, que ainda não inclui secret)
- `src/middlewares/validate.middleware.ts:11` (validação de entrada onde fica a exigência de TLS)
- Transcrição: [09:02] Marcos, [09:03] Sofia, [09:19] Sofia, [09:20] Sofia, [09:21] Bruno, [09:21] Sofia, [09:22] Diego, [09:22] Sofia, [09:23] Sofia, [09:31] Marcos, [09:44] Diego, [09:46] Sofia
