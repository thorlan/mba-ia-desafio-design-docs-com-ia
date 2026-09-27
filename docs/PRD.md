### PRD: Order Management System, Sistema de Webhooks de Notificação de Pedidos

Versão: 1.0 (em revisão)
Data: Reunião técnica de quinta-feira, 09:00 (ver [`TRANSCRICAO.md`](../TRANSCRICAO.md))
Responsável: Diego (Engenheiro Sênior, time de Plataforma)

Documentos relacionados: [RFC](RFC.md), [FDD](FDD.md), [ADRs](adrs/). Todo item tem origem na reunião (`[hh:mm] Nome`) ou no código (`caminho:linha`). Onde o template pede um campo que a reunião não respondeu, o texto diz "Não definido na reunião".

---

### Resumo

Clientes B2B do OMS querem ser avisados quando o status dos seus pedidos muda, sem precisar consultar a API de tempos em tempos ([09:00] Marcos). Esta feature permite que cada cliente cadastre endpoints de webhook, escolha os status que quer receber e receba, em menos de 10 segundos, uma notificação assinada a cada mudança ([09:02] Marcos, [09:20] Sofia, [09:33] Marcos). A entrega é resiliente a indisponibilidades de até quase 15 horas, e os eventos que esgotam as tentativas vão para uma DLQ com reprocessamento manual ([09:17] Diego, [09:18] Diego).

---

### Contexto e problema

Público-alvo
- Clientes B2B que integram com o OMS pela API; os primeiros são Atlas Comercial, MaxDistribuição e Nova Cargo ([09:00] Marcos).
- Desenvolvedores desses clientes, que integram com a documentação do portal ([09:26] Marcos, [09:40] Marcos).
- Administradores da plataforma, os únicos que reprocessam eventos da DLQ ([09:36] Sofia).

Cenários de uso chave
- Um pedido muda de status e o cliente é notificado, sem precisar ficar consultando a API ([09:00] Marcos, [09:02] Marcos).
- O cliente cadastra um webhook que só quer saber quando o pedido vira SHIPPED ou DELIVERED ([09:33] Marcos).
- O sistema do cliente fica duas horas fora do ar numa manutenção planejada, e as retentativas cobrem essa janela ([09:16] Diego).
- Uma secret vaza num log do cliente, como já aconteceu, e ele pede uma nova pela API ([09:22] Diego, [09:21] Sofia).
- O cliente consulta as últimas entregas, com sucesso ou falha ([09:34] Marcos).
- Um administrador reprocessa um evento da DLQ ([09:18] Diego).

Onde essa feature será implantada
- No OMS existente: uma API REST em Node.js e TypeScript, com um único processo HTTP e banco MySQL via Prisma (`src/server.ts:6`, `prisma/schema.prisma:5-9`). A feature entra como um módulo novo e um processo separado para o worker ([09:27] Bruno, [09:11] Diego).

Problemas priorizados
- **Integração lenta e cara:** os clientes consultam a API de pedidos de tempos em tempos, o que deixa a integração "lenta e cara pra eles" ([09:00] Marcos). Prioridade alta.
- **Risco de perder cliente:** a Atlas sinalizou que pode migrar para um concorrente se não tiver a funcionalidade até o fim do trimestre ([09:00] Marcos), e quer a entrega para o fim de novembro ([09:45] Marcos). Prioridade alta.

---

### Objetivos e métricas

| Objetivo | Métrica | Meta |
| --- | --- | --- |
| PRD-OBJ-01: notificar o cliente em "tempo real" ([09:02] Marcos) | Tempo entre a mudança de status e a entrega | Menos de 10 s ([09:02] Marcos); no pior caso, 2 s até o worker ler o evento ([09:10] Larissa) |
| PRD-OBJ-02: nunca perder uma mudança de status ([09:40] Bruno) | Mudanças de status confirmadas sem evento registrado | Zero: "Não pode ter caso de status mudar e evento não sair" ([09:40] Bruno) |
| PRD-OBJ-03: absorver indisponibilidades do cliente ([09:16] Diego) | Janela entre a primeira falha e a última tentativa | Quase 15 h ([09:17] Diego) |
| PRD-OBJ-04: entregar no prazo pedido pela Atlas ([09:45] Marcos) | Sprints até a entrega, com a revisão de segurança incluída | 3 sprints ([09:47] Larissa) |

---

### Escopo

Incluso
- Notificação a cada mudança de status de pedido, para os webhooks que assinam o novo status ([09:33] Marcos, [09:40] Bruno).
- Cadastro, listagem, edição e remoção de webhooks por customer ([09:31] Marcos, [09:33] Bruno).
- Rotação de secret com carência de 24 h ([09:21] Sofia).
- Assinatura HMAC-SHA256 e identificador único em cada envio ([09:22] Sofia, [09:25] Diego).
- Retentativas com backoff exponencial e DLQ ([09:17] Larissa).
- Histórico de entregas por webhook ([09:34] Marcos).
- Reprocessamento manual da DLQ por administradores, com registro de quem fez ([09:36] Sofia).
- Documentação para os clientes no portal do desenvolvedor ([09:40] Marcos).

Fora de escopo
- **Webhooks de entrada:** os clientes "querem receber, não mandar" ([09:02] Marcos).
- **E-mail ao cliente em falhas seguidas:** "Email tá fora de escopo dessa fase. Talvez próxima fase, depois que a gente medir o impacto." ([09:37] Larissa).
- **Painel visual para o cliente:** "Não, agora não. Só endpoints. Painel é projeto separado do time de frontend." ([09:40] Larissa).
- **Rate limiting de envio:** adiado; fica como "observar e decidir depois" ([09:39] Larissa).
- **Vários workers em paralelo:** "isso é problema do futuro, não agora" ([09:13] Diego).
- **Arquivamento de eventos entregues:** "fora do escopo dessa feature" ([09:08] Diego).
- **Restrição de papel no cadastro de webhooks:** qualquer papel autenticado pode gerenciar webhooks "Por enquanto" ([09:37] Sofia).
- **Truncar payloads grandes:** descartado; acima do limite, o evento não é enviado ([09:23] Sofia, [09:24] Larissa).

---

### Requisitos funcionais

#### PRD-FR-01 Notificar mudanças de status
A cada mudança de status de um pedido, o sistema notifica os webhooks do customer que assinam o novo status ([09:00] Marcos, [09:34] Bruno).

**Fluxo principal**
- Um usuário muda o status de um pedido ([09:40] Bruno).
- Na mesma transação, o sistema registra o evento na outbox ([09:06] Diego, [09:40] Bruno), já com o payload montado ([09:52] Larissa).
- Um worker separado lê os eventos pendentes a cada 2 s e faz a chamada HTTP ao cliente ([09:06] Diego, [09:10] Larissa).

**Fluxos alternativos e exceções**
- Nenhum webhook do customer assina o novo status: nenhum evento é registrado ([09:34] Bruno).
- Falha ao registrar o evento: a mudança de status sofre rollback ([09:40] Bruno).

**Erros previstos**
- Falha de gravação do evento, que desfaz a mudança de status ([09:40] Bruno).

**Prioridade:** alta

---

#### PRD-FR-02 Cadastrar webhook
O cliente cadastra um webhook informando a URL e os status que quer receber; a plataforma gera a secret e a devolve na criação ([09:31] Marcos).

**Fluxo principal**
- O usuário autenticado envia a URL, a lista de status e o customer ([09:31] Marcos, [09:32] Larissa).
- A plataforma gera uma secret única para o endpoint ([09:21] Sofia, [09:31] Marcos).
- A plataforma devolve o webhook criado com a secret ([09:31] Marcos).

**Fluxos alternativos e exceções**
- O customer é passado no corpo ou no caminho da requisição, não vem do token ([09:32] Larissa).

**Erros previstos**
- URL `http`: recusada com erro de validação ([09:23] Sofia).

**Prioridade:** alta

---

#### PRD-FR-03 Filtrar eventos por status
Cada webhook define a lista de status que quer receber, e só esses geram notificação ([09:33] Marcos).

**Fluxo principal**
- O cliente informa a lista de status, por exemplo SHIPPED e DELIVERED ([09:33] Marcos).
- O filtro é aplicado na inserção do evento: só os webhooks que assinam o novo status recebem evento ([09:34] Bruno).

**Fluxos alternativos e exceções**
- Se nenhum webhook do customer quer aquele status, o evento nem é inserido ([09:34] Bruno).

**Erros previstos**
- Não definido na reunião.

**Prioridade:** alta

---

#### PRD-FR-04 Listar webhooks de um customer
O cliente lista os webhooks de um customer ([09:33] Bruno).

**Fluxo principal**
- O usuário autenticado pede a lista de webhooks de um customer ([09:33] Bruno).

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Não definido na reunião.

**Prioridade:** media

---

#### PRD-FR-05 Editar webhook
O cliente edita um webhook ([09:33] Bruno), incluindo os eventos que quer receber ([09:33] Bruno) e o estado ativo ([09:21] Bruno).

**Fluxo principal**
- O usuário envia a alteração de um webhook ([09:33] Bruno).

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Webhook inexistente: `WEBHOOK_NOT_FOUND` ([09:28] Bruno).
- URL `http`: recusada com erro de validação ([09:23] Sofia).

**Prioridade:** media

---

#### PRD-FR-06 Remover webhook
O cliente remove um webhook ([09:33] Bruno).

**Fluxo principal**
- O usuário pede a remoção de um webhook ([09:33] Bruno).

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Webhook inexistente: `WEBHOOK_NOT_FOUND` ([09:28] Bruno).

**Prioridade:** media

---

#### PRD-FR-07 Rotacionar secret
O cliente pede uma nova secret pela API; a antiga fica válida por 24 h em paralelo e depois é invalidada ([09:21] Sofia).

**Fluxo principal**
- O usuário pede uma nova secret pela API ([09:21] Sofia).
- A secret antiga continua válida por 24 h, para o cliente migrar os sistemas dele ([09:21] Sofia).
- Depois disso, a antiga deixa de valer ([09:21] Sofia).

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Webhook inexistente: `WEBHOOK_NOT_FOUND` ([09:28] Bruno).

**Prioridade:** alta

---

#### PRD-FR-08 Assinar cada envio
Todo envio carrega uma assinatura HMAC-SHA256 do corpo, com a secret do endpoint, para o cliente validar que a requisição veio da plataforma e que o payload não foi adulterado ([09:19] Sofia, [09:22] Sofia).

**Fluxo principal**
- A plataforma assina o corpo com a secret do webhook ([09:20] Sofia, [09:22] Sofia).
- A assinatura vai no header `X-Signature` ([09:20] Sofia).
- O cliente verifica do lado dele ([09:20] Sofia).

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Não definido na reunião.

**Prioridade:** alta

---

#### PRD-FR-09 Identificar cada evento para deduplicação
Todo evento tem um UUID gerado quando entra na outbox, enviado no header `X-Event-Id` e no payload, para o cliente deduplicar ([09:25] Diego, [09:43] Diego).

**Fluxo principal**
- O UUID é gerado quando o evento entra na outbox ([09:25] Diego, [09:51] Larissa).
- Ele vai no header `X-Event-Id` ([09:44] Diego) e no campo `event_id` do payload ([09:43] Diego).
- Se o cliente receber o mesmo evento duas vezes, deduplica pelo identificador ([09:25] Diego).

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Não definido na reunião.

**Prioridade:** alta

---

#### PRD-FR-10 Retentar entregas com falha
Uma entrega com falha é retentada 5 vezes, com backoff de 1 min, 5 min, 30 min, 2 h e 12 h ([09:17] Larissa).

**Fluxo principal**
- A chamada ao cliente falha, ou ele não responde em 10 s ([09:42] Diego).
- O evento é marcado para retry ([09:42] Diego), com o próximo intervalo da progressão ([09:17] Diego).

**Fluxos alternativos e exceções**
- Depois do teto de tentativas, o evento é considerado falha permanente e vai para a DLQ ([09:15] Diego).

**Erros previstos**
- Cliente que não responde em 10 s ([09:42] Diego).

**Prioridade:** alta

---

#### PRD-FR-11 Guardar eventos esgotados na DLQ
Um evento que esgotou as tentativas vai para uma DLQ em tabela separada, com o payload, o motivo da falha e o timestamp ([09:17] Larissa, [09:18] Diego).

**Fluxo principal**
- As tentativas se esgotam ([09:15] Diego).
- O evento é gravado na DLQ com payload, motivo e timestamp ([09:18] Diego).

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Não definido na reunião.

**Prioridade:** alta

---

#### PRD-FR-12 Reprocessar evento da DLQ
Um administrador reprocessa manualmente um evento da DLQ, e a operação registra quem a fez ([09:18] Diego, [09:36] Sofia).

**Fluxo principal**
- Um administrador pede o replay de um evento da DLQ ([09:18] Diego, [09:35] Diego).
- O sistema exige o papel de administrador, com o `requireRole` existente ([09:36] Larissa).
- O evento é recolocado na outbox como pendente ([09:18] Diego), e o sistema loga quem fez o replay ([09:36] Sofia).

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Usuário sem o papel de administrador: acesso negado ([09:36] Sofia).

**Prioridade:** alta

---

#### PRD-FR-13 Consultar histórico de entregas
O cliente vê as últimas 100 entregas de um webhook, com sucesso ou falha, payload, resposta e tempo de resposta ([09:34] Marcos).

**Fluxo principal**
- O usuário pede o histórico de um webhook ([09:34] Marcos).
- O sistema devolve as últimas 100 entregas ([09:34] Marcos).

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Webhook inexistente: `WEBHOOK_NOT_FOUND` ([09:28] Bruno).

**Prioridade:** media

---

### Requisitos não funcionais

Performance
- PRD-NFR-01: entrega em menos de 10 s ([09:02] Marcos); no pior caso, 2 s até o worker ler o evento ([09:10] Larissa).
- PRD-NFR-02: timeout de 10 s na chamada ao cliente ([09:42] Diego).

Disponibilidade
- PRD-NFR-03: o worker roda em processo separado, para que um restart da API não o derrube ([09:11] Diego).
- PRD-NFR-04: indisponibilidades do cliente de até quase 15 h são cobertas pelas retentativas ([09:17] Diego).

Segurança e autorização
- PRD-NFR-05: só URLs `https` ([09:23] Sofia).
- PRD-NFR-06: secret única por endpoint, rotacionável com carência de 24 h ([09:21] Sofia).
- PRD-NFR-07: rotas de cadastro autenticadas; replay exige o papel de administrador e registra quem fez ([09:36] Sofia, [09:36] Larissa, [09:48] Larissa).
- PRD-NFR-08: revisão de segurança do código antes do deploy, com foco em HMAC e geração de secret ([09:46] Sofia).

Observabilidade
- PRD-NFR-09: logs com o Pino já existente, sem biblioteca nova ([09:29] Bruno).
- PRD-NFR-10: histórico de entregas com resultado, payload, resposta e tempo de resposta ([09:34] Marcos).

Confiabilidade e integridade de dados
- PRD-NFR-11: evento registrado na mesma transação da mudança de status ([09:06] Diego, [09:40] Bruno).
- PRD-NFR-12: entrega at-least-once ([09:24] Diego).
- PRD-NFR-13: payload renderizado na inserção, refletindo o pedido no momento da mudança ([09:52] Larissa).
- PRD-NFR-14: ordem garantida só por pedido e enquanto houver um único worker ([09:13] Larissa).

Compatibilidade e portabilidade
- PRD-NFR-15: rotas sob o prefixo `/api/v1`, como o resto da API (`src/app.ts:67`).
- PRD-NFR-16: mesmo MySQL e mesma stack, sem infraestrutura nova ([09:07] Diego, [09:11] Diego).
- PRD-NFR-17: payload de no máximo 64 KB, com erro se ultrapassar ([09:24] Larissa).

Compliance
- PRD-NFR-18: registro de quem fez cada replay, para auditoria ([09:36] Sofia).

Acessibilidade no frontend consumidor
- Não se aplica: a feature só expõe endpoints, e o painel está fora de escopo ([09:40] Larissa).

---

### Arquitetura e abordagem

Abordagem
- Padrão outbox no MySQL existente, com um worker separado que lê a tabela e faz as chamadas HTTP ([09:06] Diego, [09:07] Diego). Visão completa no [RFC](RFC.md).

Componentes
- Módulo de webhooks com a mesma estrutura dos módulos atuais ([09:27] Bruno).
- Outbox, DLQ e configuração de webhooks no MySQL ([09:06] Diego, [09:18] Diego, [09:21] Bruno).
- Worker em processo separado, com o mesmo banco e a mesma stack ([09:11] Diego).

Integrações
- Serviço de pedidos: a inserção na outbox acontece dentro da transação de mudança de status ([09:40] Bruno, `src/modules/orders/order.service.ts:126`).
- Endpoints dos clientes, fora da infraestrutura da plataforma ([09:19] Sofia).
- Portal do desenvolvedor, com a documentação de integração ([09:40] Marcos).

### Decisões e trade-offs

#### Decisão: Outbox transacional no MySQL existente ([ADR-001](adrs/ADR-001-outbox-transacional-no-mysql.md))
- **Justificativa:** se a transação principal commitou, o evento foi registrado, e se deu rollback, o evento some junto ([09:06] Diego); sem subir infraestrutura nova ([09:07] Diego).
- **Trade-off:** acrescenta trabalho a uma transação que já é pesada ([09:04] Bruno).

#### Decisão: Worker em processo separado com polling ([ADR-002](adrs/ADR-002-worker-em-processo-separado-com-polling.md))
- **Justificativa:** o MySQL não notifica processo externo ([09:09] Diego), e o worker não pode cair quando a API reinicia ([09:11] Diego).
- **Trade-off:** latência de 2 s no pior caso ([09:10] Larissa) e ordem só por pedido enquanto houver um único worker ([09:13] Larissa).

#### Decisão: Retry com backoff exponencial e DLQ ([ADR-003](adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md))
- **Justificativa:** 3 tentativas "é pouco" ([09:16] Diego), e retry indefinido deixa evento "pendurado pra sempre se o cliente sumiu" ([09:15] Diego).
- **Trade-off:** acima de 15 h de indisponibilidade, o problema é do cliente ([09:17] Marcos), e o reprocessamento é manual ([09:18] Diego).

#### Decisão: HMAC-SHA256 com secret por endpoint e rotação ([ADR-004](adrs/ADR-004-autenticacao-hmac-sha256-com-secret-por-endpoint.md))
- **Justificativa:** é o padrão de mercado ([09:20] Sofia), e secret por endpoint evita que "se vaza uma, vaza tudo" ([09:21] Sofia).
- **Trade-off:** o cliente precisa migrar para a nova secret em até 24 h depois de uma rotação ([09:21] Sofia).

#### Decisão: Entrega at-least-once com X-Event-Id ([ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md))
- **Justificativa:** padrão de mercado, e exactly-once "exigiria coordenação dos dois lados e fica muito mais complexo" ([09:25] Diego).
- **Trade-off:** "Isso joga responsabilidade pro cliente" ([09:25] Sofia).

#### Decisão: Reuso dos padrões existentes do projeto ([ADR-006](adrs/ADR-006-reuso-dos-padroes-do-projeto.md))
- **Justificativa:** "Não vamos botar nada novo", e o middleware de erro já trata os erros do módulo ([09:29] Bruno).
- **Trade-off:** o middleware de validação transforma todo erro de schema em `VALIDATION_ERROR` (`src/middlewares/validate.middleware.ts:31`), e não num código `WEBHOOK_`.

---

### Dependências

#### Organizacional: revisão de segurança
Pelo menos dois dias úteis para a Sofia revisar o código de segurança antes do deploy, em especial HMAC e geração de secret ([09:46] Sofia), incluídos nas três sprints ([09:47] Larissa).

#### Organizacional: documentação no portal do desenvolvedor
O Marcos documenta no portal como integrar via API ([09:40] Marcos), com destaque para a deduplicação ([09:26] Marcos).

#### Externa: confirmação de prazo com a Atlas
O Marcos confirma o prazo com a Atlas ([09:47] Marcos) e atualiza os clientes ([09:49] Marcos).

#### Externa: preparo dos clientes
O cliente precisa de endpoint `https` ([09:23] Sofia), verificar a assinatura do lado dele ([09:20] Sofia) e estar preparado para receber o mesmo evento duas vezes ([09:24] Diego).

#### Técnica: processo do worker
O worker é um processo separado, conectado ao mesmo banco ([09:11] Diego, [09:30] Bruno), e carrega a mesma validação de ambiente da API (`src/config/env.ts:3-10`).

---

### Riscos e mitigação

#### PRD-RISCO-01: A entrega atrasa e a Atlas migra para um concorrente
- **Probabilidade:** baixa (evidência: depois da estimativa de três sprints, "Atlas vai gostar", [09:47] Marcos)
- **Impacto:** perda de um cliente B2B, que ameaçou migrar ([09:00] Marcos).
- **Mitigação:**
  - Escopo enxuto: e-mail, painel e rate limiting fora desta fase ([09:48] Larissa).
  - Confirmação do prazo com a Atlas ([09:47] Marcos).
- **Plano de contingência:** Não definido na reunião.

#### PRD-RISCO-02: Um cliente não deduplica e processa o mesmo evento duas vezes
- **Probabilidade:** media (evidência: "Pode acontecer de o cliente receber o mesmo evento duas vezes", [09:24] Diego; "Isso joga responsabilidade pro cliente", [09:25] Sofia)
- **Impacto:** o cliente recebe o mesmo evento duas vezes e precisa deduplicar do lado dele ([09:25] Diego).
- **Mitigação:**
  - Documentação em destaque no portal ([09:26] Marcos).
- **Plano de contingência:** Não definido na reunião.

#### PRD-RISCO-03: Uma secret vaza
- **Probabilidade:** media (evidência: "A gente já teve cliente que vazou secret em log de aplicação dele uma vez", [09:22] Diego)
- **Impacto:** "se vaza uma, vaza tudo" com secret global ([09:21] Sofia); com secret por endpoint, o vazamento fica restrito àquele endpoint.
- **Mitigação:**
  - Secret única por endpoint ([09:21] Sofia).
  - Revisão de segurança antes do deploy ([09:46] Sofia).
- **Plano de contingência:** rotação da secret com carência de 24 h ([09:21] Sofia).

#### PRD-RISCO-04: Acesso indevido ao cadastro de webhooks e ao replay
- **Probabilidade:** media (evidência: o cadastro fica aberto a qualquer papel autenticado, [09:37] Sofia; não há vínculo entre usuário e customer, `prisma/schema.prisma:25-54`; e o registro de usuário aceita o papel de administrador, `src/modules/auth/auth.schemas.ts:7`)
- **Impacto:** um usuário gerencia webhooks de qualquer customer (`prisma/schema.prisma:25-54`), ou se registra com o papel de administrador (`src/modules/auth/auth.schemas.ts:7`) e faz replay.
- **Mitigação:**
  - Replay restrito ao papel de administrador ([09:36] Larissa), com registro de quem fez ([09:36] Sofia).
- **Plano de contingência:** "Mais pra frente a gente pode endurecer." ([09:37] Sofia).

#### PRD-RISCO-05: A transação de mudança de status fica mais lenta
- **Probabilidade:** media (evidência: "A transação de mudança de status hoje já é pesada", [09:04] Bruno)
- **Impacto:** mais trabalho numa transação que já é pesada ([09:04] Bruno).
- **Mitigação:**
  - Filtrar na inserção: se nenhum webhook quer o status, nem insere ([09:34] Bruno).
- **Plano de contingência:** Não definido na reunião.

#### PRD-RISCO-06: Um cliente recebe uma rajada de chamadas
- **Probabilidade:** baixa (evidência: "A gente observa e implementa se virar problema", [09:39] Diego)
- **Impacto:** 50 pedidos mudando de status em um minuto viram 50 chamadas ao cliente ([09:38] Diego).
- **Mitigação:**
  - Observar e decidir depois ([09:39] Larissa).
- **Plano de contingência:** implementar rate limiting "se virar problema" ([09:39] Diego).

#### PRD-RISCO-07: Um evento passa de 64 KB
- **Probabilidade:** baixa (evidência: "Nenhum evento nosso vai chegar perto disso", [09:24] Diego)
- **Impacto:** o evento não é enviado e gera erro ([09:23] Sofia, [09:24] Larissa).
- **Mitigação:**
  - Payload sem os itens do pedido, "pra não inflar" ([09:43] Diego).
- **Plano de contingência:** Não definido na reunião.

---

### Critérios de aceitação
Checklist objetivo que define se a feature está pronta. Os critérios técnicos estão na seção 9 do [FDD](FDD.md).

- PRD-CA-01: toda mudança de status para um status assinado gera um evento na mesma transação, e uma mudança desfeita não gera evento ([09:06] Diego, [09:40] Bruno).
- PRD-CA-02: a entrega acontece em menos de 10 s após a mudança de status ([09:02] Marcos).
- PRD-CA-03: mudança para um status que nenhum webhook do customer assina não gera evento ([09:34] Bruno).
- PRD-CA-04: o cliente cadastra, lista, edita e remove webhooks pela API ([09:33] Bruno), e URL `http` é recusada ([09:23] Sofia).
- PRD-CA-05: todo envio traz os headers `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type` ([09:44] Diego, [09:44] Sofia).
- PRD-CA-06: depois de uma rotação, a secret antiga vale por 24 h e depois deixa de valer ([09:21] Sofia).
- PRD-CA-07: uma entrega que sempre falha é retentada 5 vezes, em 1 min, 5 min, 30 min, 2 h e 12 h, e termina na DLQ ([09:17] Larissa).
- PRD-CA-08: só o papel de administrador consegue fazer replay, e cada replay registra quem fez ([09:36] Sofia).
- PRD-CA-09: o histórico mostra as últimas 100 entregas de um webhook ([09:34] Marcos).
- PRD-CA-10: a revisão de segurança da Sofia foi feita antes do deploy ([09:49] Sofia).
- PRD-CA-11: a documentação de integração está publicada no portal ([09:40] Marcos).

---

### Testes e validação

Tipos de teste obrigatórios
- Testes ponta a ponta, incluídos na estimativa junto com a integração no serviço de pedidos ([09:46] Larissa), no padrão de testes do projeto, com Vitest e Supertest contra a API (`tests/orders.test.ts`, `package.json:17`).
- Revisão de segurança do código antes do deploy, com foco em HMAC e geração de secret ([09:46] Sofia).

Estratégia de validação
- A revisão de segurança entra no fim das três sprints ([09:47] Larissa).
- Demais etapas de validação: não definidas na reunião.
