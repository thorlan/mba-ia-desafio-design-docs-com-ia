# Tracker de Rastreabilidade

Cada linha liga um item dos documentos do pacote à sua origem: uma fala da reunião (`TRANSCRICAO`, localizada por `[hh:mm] Nome`) ou um arquivo do código (`CODIGO`, localizado pelo caminho). Quando um item cita várias falas, a linha usa a que o sustenta; as demais estão no próprio documento. As questões em aberto do RFC que a reunião não discutiu (RFC-QB) apontam para a fala em que o tema aparece. O que os documentos marcam como "Não definido na reunião" não é rastreado, porque não tem origem.

**Cobertura:** 184 de 184 itens identificados nos documentos (100%). **Fontes:** 172 linhas com `TRANSCRICAO` (93%) e 12 com `CODIGO`. Gerado com o prompt [`process/prompts/08-tracker-adaptado.md`](../process/prompts/08-tracker-adaptado.md).

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-OBJ-01 | `docs/PRD.md` | Objetivo | Notificar o cliente em "tempo real" | TRANSCRICAO | [09:02] Marcos |
| PRD-OBJ-02 | `docs/PRD.md` | Objetivo | Nunca perder uma mudança de status | TRANSCRICAO | [09:40] Bruno |
| PRD-OBJ-03 | `docs/PRD.md` | Objetivo | Absorver indisponibilidades do cliente | TRANSCRICAO | [09:16] Diego |
| PRD-OBJ-04 | `docs/PRD.md` | Objetivo | Entregar no prazo pedido pela Atlas | TRANSCRICAO | [09:45] Marcos |
| PRD-ESC-01 | `docs/PRD.md` | Escopo | Notificação a cada mudança de status de pedido, para os webhooks que assinam o novo status | TRANSCRICAO | [09:33] Marcos |
| PRD-ESC-02 | `docs/PRD.md` | Escopo | Cadastro, listagem, edição e remoção de webhooks por customer | TRANSCRICAO | [09:31] Marcos |
| PRD-ESC-03 | `docs/PRD.md` | Escopo | Rotação de secret com carência de 24 h | TRANSCRICAO | [09:21] Sofia |
| PRD-ESC-04 | `docs/PRD.md` | Escopo | Assinatura HMAC-SHA256 e identificador único em cada envio | TRANSCRICAO | [09:22] Sofia |
| PRD-ESC-05 | `docs/PRD.md` | Escopo | Retentativas com backoff exponencial e DLQ | TRANSCRICAO | [09:17] Larissa |
| PRD-ESC-06 | `docs/PRD.md` | Escopo | Histórico de entregas por webhook | TRANSCRICAO | [09:34] Marcos |
| PRD-ESC-07 | `docs/PRD.md` | Escopo | Reprocessamento manual da DLQ por administradores, com registro de quem fez | TRANSCRICAO | [09:36] Sofia |
| PRD-ESC-08 | `docs/PRD.md` | Escopo | Documentação para os clientes no portal do desenvolvedor | TRANSCRICAO | [09:40] Marcos |
| PRD-FORA-01 | `docs/PRD.md` | Fora de Escopo | Webhooks de entrada | TRANSCRICAO | [09:02] Marcos |
| PRD-FORA-02 | `docs/PRD.md` | Fora de Escopo | E-mail ao cliente em falhas seguidas | TRANSCRICAO | [09:37] Larissa |
| PRD-FORA-03 | `docs/PRD.md` | Fora de Escopo | Painel visual para o cliente | TRANSCRICAO | [09:40] Larissa |
| PRD-FORA-04 | `docs/PRD.md` | Fora de Escopo | Rate limiting de envio | TRANSCRICAO | [09:39] Larissa |
| PRD-FORA-05 | `docs/PRD.md` | Fora de Escopo | Vários workers em paralelo | TRANSCRICAO | [09:13] Diego |
| PRD-FORA-06 | `docs/PRD.md` | Fora de Escopo | Arquivamento de eventos entregues | TRANSCRICAO | [09:08] Diego |
| PRD-FORA-07 | `docs/PRD.md` | Fora de Escopo | Restrição de papel no cadastro de webhooks | TRANSCRICAO | [09:37] Sofia |
| PRD-FORA-08 | `docs/PRD.md` | Fora de Escopo | Truncar payloads grandes | TRANSCRICAO | [09:23] Sofia |
| PRD-FR-01 | `docs/PRD.md` | Requisito Funcional | Notificar mudanças de status | TRANSCRICAO | [09:00] Marcos |
| PRD-FR-02 | `docs/PRD.md` | Requisito Funcional | Cadastrar webhook | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-03 | `docs/PRD.md` | Requisito Funcional | Filtrar eventos por status | TRANSCRICAO | [09:33] Marcos |
| PRD-FR-04 | `docs/PRD.md` | Requisito Funcional | Listar webhooks de um customer | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-05 | `docs/PRD.md` | Requisito Funcional | Editar webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-06 | `docs/PRD.md` | Requisito Funcional | Remover webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-07 | `docs/PRD.md` | Requisito Funcional | Rotacionar secret | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-08 | `docs/PRD.md` | Requisito Funcional | Assinar cada envio | TRANSCRICAO | [09:19] Sofia |
| PRD-FR-09 | `docs/PRD.md` | Requisito Funcional | Identificar cada evento para deduplicação | TRANSCRICAO | [09:25] Diego |
| PRD-FR-10 | `docs/PRD.md` | Requisito Funcional | Retentar entregas com falha | TRANSCRICAO | [09:17] Larissa |
| PRD-FR-11 | `docs/PRD.md` | Requisito Funcional | Guardar eventos esgotados na DLQ | TRANSCRICAO | [09:17] Larissa |
| PRD-FR-12 | `docs/PRD.md` | Requisito Funcional | Reprocessar evento da DLQ | TRANSCRICAO | [09:18] Diego |
| PRD-FR-13 | `docs/PRD.md` | Requisito Funcional | Consultar histórico de entregas | TRANSCRICAO | [09:34] Marcos |
| PRD-NFR-01 | `docs/PRD.md` | Requisito Não Funcional | Entrega em menos de 10 s; no pior caso, 2 s até o worker ler o evento | TRANSCRICAO | [09:02] Marcos |
| PRD-NFR-02 | `docs/PRD.md` | Requisito Não Funcional | Timeout de 10 s na chamada ao cliente | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-03 | `docs/PRD.md` | Requisito Não Funcional | O worker roda em processo separado, para que um restart da API não o derrube | TRANSCRICAO | [09:11] Diego |
| PRD-NFR-04 | `docs/PRD.md` | Requisito Não Funcional | Indisponibilidades do cliente de até quase 15 h são cobertas pelas retentativas | TRANSCRICAO | [09:17] Diego |
| PRD-NFR-05 | `docs/PRD.md` | Requisito Não Funcional | Só URLs `https` | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-06 | `docs/PRD.md` | Requisito Não Funcional | Secret única por endpoint, rotacionável com carência de 24 h | TRANSCRICAO | [09:21] Sofia |
| PRD-NFR-07 | `docs/PRD.md` | Requisito Não Funcional | Rotas de cadastro autenticadas; replay exige o papel de administrador e registra quem fez | TRANSCRICAO | [09:36] Sofia |
| PRD-NFR-08 | `docs/PRD.md` | Requisito Não Funcional | Revisão de segurança do código antes do deploy, com foco em HMAC e geração de secret | TRANSCRICAO | [09:46] Sofia |
| PRD-NFR-09 | `docs/PRD.md` | Requisito Não Funcional | Logs com o Pino já existente, sem biblioteca nova | TRANSCRICAO | [09:29] Bruno |
| PRD-NFR-10 | `docs/PRD.md` | Requisito Não Funcional | Histórico de entregas com resultado, payload, resposta e tempo de resposta | TRANSCRICAO | [09:34] Marcos |
| PRD-NFR-11 | `docs/PRD.md` | Requisito Não Funcional | Evento registrado na mesma transação da mudança de status | TRANSCRICAO | [09:06] Diego |
| PRD-NFR-12 | `docs/PRD.md` | Requisito Não Funcional | Entrega at-least-once | TRANSCRICAO | [09:24] Diego |
| PRD-NFR-13 | `docs/PRD.md` | Requisito Não Funcional | Payload renderizado na inserção, refletindo o pedido no momento da mudança | TRANSCRICAO | [09:52] Larissa |
| PRD-NFR-14 | `docs/PRD.md` | Requisito Não Funcional | Ordem garantida só por pedido e enquanto houver um único worker | TRANSCRICAO | [09:13] Larissa |
| PRD-NFR-15 | `docs/PRD.md` | Requisito Não Funcional | Rotas sob o prefixo `/api/v1`, como o resto da API | CODIGO | `src/app.ts` |
| PRD-NFR-16 | `docs/PRD.md` | Requisito Não Funcional | Mesmo MySQL e mesma stack, sem infraestrutura nova | TRANSCRICAO | [09:07] Diego |
| PRD-NFR-17 | `docs/PRD.md` | Requisito Não Funcional | Payload de no máximo 64 KB, com erro se ultrapassar | TRANSCRICAO | [09:24] Larissa |
| PRD-NFR-18 | `docs/PRD.md` | Requisito Não Funcional | Registro de quem fez cada replay, para auditoria | TRANSCRICAO | [09:36] Sofia |
| PRD-DEC-01 | `docs/PRD.md` | Decisão | Outbox transacional no MySQL existente | TRANSCRICAO | [09:06] Diego |
| PRD-DEC-02 | `docs/PRD.md` | Decisão | Worker em processo separado com polling | TRANSCRICAO | [09:09] Diego |
| PRD-DEC-03 | `docs/PRD.md` | Decisão | Retry com backoff exponencial e DLQ | TRANSCRICAO | [09:16] Diego |
| PRD-DEC-04 | `docs/PRD.md` | Decisão | HMAC-SHA256 com secret por endpoint e rotação | TRANSCRICAO | [09:20] Sofia |
| PRD-DEC-05 | `docs/PRD.md` | Decisão | Entrega at-least-once com X-Event-Id | TRANSCRICAO | [09:25] Diego |
| PRD-DEC-06 | `docs/PRD.md` | Decisão | Reuso dos padrões existentes do projeto | TRANSCRICAO | [09:29] Bruno |
| PRD-DEP-01 | `docs/PRD.md` | Dependência | Organizacional: revisão de segurança | TRANSCRICAO | [09:46] Sofia |
| PRD-DEP-02 | `docs/PRD.md` | Dependência | Organizacional: documentação no portal do desenvolvedor | TRANSCRICAO | [09:40] Marcos |
| PRD-DEP-03 | `docs/PRD.md` | Dependência | Externa: confirmação de prazo com a Atlas | TRANSCRICAO | [09:47] Marcos |
| PRD-DEP-04 | `docs/PRD.md` | Dependência | Externa: preparo dos clientes | TRANSCRICAO | [09:23] Sofia |
| PRD-DEP-05 | `docs/PRD.md` | Dependência | Técnica: processo do worker | TRANSCRICAO | [09:11] Diego |
| PRD-RISCO-01 | `docs/PRD.md` | Risco | A entrega atrasa e a Atlas migra para um concorrente | TRANSCRICAO | [09:00] Marcos |
| PRD-RISCO-02 | `docs/PRD.md` | Risco | Um cliente não deduplica e processa o mesmo evento duas vezes | TRANSCRICAO | [09:24] Diego |
| PRD-RISCO-03 | `docs/PRD.md` | Risco | Uma secret vaza | TRANSCRICAO | [09:22] Diego |
| PRD-RISCO-04 | `docs/PRD.md` | Risco | Acesso indevido ao cadastro de webhooks e ao replay | TRANSCRICAO | [09:37] Sofia |
| PRD-RISCO-05 | `docs/PRD.md` | Risco | A transação de mudança de status fica mais lenta | TRANSCRICAO | [09:04] Bruno |
| PRD-RISCO-06 | `docs/PRD.md` | Risco | Um cliente recebe uma rajada de chamadas | TRANSCRICAO | [09:38] Diego |
| PRD-RISCO-07 | `docs/PRD.md` | Risco | Um evento passa de 64 KB | TRANSCRICAO | [09:24] Diego |
| PRD-CA-01 | `docs/PRD.md` | Critério de Aceite | Toda mudança de status para um status assinado gera um evento na mesma transação, e uma mudança desfeita não... | TRANSCRICAO | [09:06] Diego |
| PRD-CA-02 | `docs/PRD.md` | Critério de Aceite | A entrega acontece em menos de 10 s após a mudança de status | TRANSCRICAO | [09:02] Marcos |
| PRD-CA-03 | `docs/PRD.md` | Critério de Aceite | Mudança para um status que nenhum webhook do customer assina não gera evento | TRANSCRICAO | [09:34] Bruno |
| PRD-CA-04 | `docs/PRD.md` | Critério de Aceite | O cliente cadastra, lista, edita e remove webhooks pela API, e URL `http` é recusada | TRANSCRICAO | [09:33] Bruno |
| PRD-CA-05 | `docs/PRD.md` | Critério de Aceite | Todo envio traz os headers `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type` | TRANSCRICAO | [09:44] Diego |
| PRD-CA-06 | `docs/PRD.md` | Critério de Aceite | Depois de uma rotação, a secret antiga vale por 24 h e depois deixa de valer | TRANSCRICAO | [09:21] Sofia |
| PRD-CA-07 | `docs/PRD.md` | Critério de Aceite | Uma entrega que sempre falha é retentada 5 vezes, em 1 min, 5 min, 30 min, 2 h e 12 h, e termina na DLQ | TRANSCRICAO | [09:17] Larissa |
| PRD-CA-08 | `docs/PRD.md` | Critério de Aceite | Só o papel de administrador consegue fazer replay, e cada replay registra quem fez | TRANSCRICAO | [09:36] Sofia |
| PRD-CA-09 | `docs/PRD.md` | Critério de Aceite | O histórico mostra as últimas 100 entregas de um webhook | TRANSCRICAO | [09:34] Marcos |
| PRD-CA-10 | `docs/PRD.md` | Critério de Aceite | A revisão de segurança da Sofia foi feita antes do deploy | TRANSCRICAO | [09:49] Sofia |
| PRD-CA-11 | `docs/PRD.md` | Critério de Aceite | A documentação de integração está publicada no portal | TRANSCRICAO | [09:40] Marcos |
| RFC-COMP-01 | `docs/RFC.md` | Decisão | Outbox transacional no MySQL | TRANSCRICAO | [09:06] Diego |
| RFC-COMP-02 | `docs/RFC.md` | Decisão | Worker em processo separado | TRANSCRICAO | [09:11] Diego |
| RFC-COMP-03 | `docs/RFC.md` | Decisão | Retry com backoff e DLQ | TRANSCRICAO | [09:17] Larissa |
| RFC-COMP-04 | `docs/RFC.md` | Decisão | Autenticação HMAC-SHA256 | TRANSCRICAO | [09:22] Sofia |
| RFC-COMP-05 | `docs/RFC.md` | Decisão | Entrega at-least-once | TRANSCRICAO | [09:25] Diego |
| RFC-COMP-06 | `docs/RFC.md` | Decisão | Módulo no padrão do projeto | TRANSCRICAO | [09:30] Larissa |
| RFC-ALT-01 | `docs/RFC.md` | Alternativa | Disparo síncrono na mudança de status | TRANSCRICAO | [09:03] Larissa |
| RFC-ALT-02 | `docs/RFC.md` | Alternativa | Fila externa (Redis Streams ou similar) | TRANSCRICAO | [09:07] Larissa |
| RFC-ALT-03 | `docs/RFC.md` | Alternativa | Trigger no banco para acordar o worker | TRANSCRICAO | [09:09] Bruno |
| RFC-ALT-04 | `docs/RFC.md` | Alternativa | Worker dentro do processo da API | TRANSCRICAO | [09:11] Diego |
| RFC-ALT-05 | `docs/RFC.md` | Alternativa | Apenas 3 tentativas | TRANSCRICAO | [09:16] Bruno |
| RFC-ALT-06 | `docs/RFC.md` | Alternativa | Secret global da plataforma | TRANSCRICAO | [09:21] Sofia |
| RFC-ALT-07 | `docs/RFC.md` | Alternativa | Exactly-once | TRANSCRICAO | [09:25] Diego |
| RFC-QA-01 | `docs/RFC.md` | Questão em Aberto | Rate limiting de envio: um cliente com 50 pedidos mudando em um minuto recebe 50 chamadas | TRANSCRICAO | [09:38] Diego |
| RFC-QA-02 | `docs/RFC.md` | Questão em Aberto | Escalar para vários workers, perdendo a ordem por pedido | TRANSCRICAO | [09:13] Bruno |
| RFC-QA-03 | `docs/RFC.md` | Questão em Aberto | Endurecer a autorização do cadastro de webhooks, hoje aberto a qualquer papel autenticado | TRANSCRICAO | [09:36] Marcos |
| RFC-QA-04 | `docs/RFC.md` | Questão em Aberto | Avisar o cliente por e-mail quando as entregas falham seguidamente | TRANSCRICAO | [09:37] Marcos |
| RFC-QA-05 | `docs/RFC.md` | Questão em Aberto | Arquivar os eventos já entregues | TRANSCRICAO | [09:08] Diego |
| RFC-QB-01 | `docs/RFC.md` | Questão em Aberto | O que acontece com os eventos já gravados quando o webhook é desativado ou removido | TRANSCRICAO | [09:21] Bruno |
| RFC-QB-02 | `docs/RFC.md` | Questão em Aberto | Qual o teto de latência aceitável para o acréscimo na transação de mudança de status | TRANSCRICAO | [09:04] Bruno |
| RFC-QB-03 | `docs/RFC.md` | Questão em Aberto | Como recuperar eventos presos em processamento quando o worker cai | TRANSCRICAO | [09:11] Diego |
| RFC-QB-04 | `docs/RFC.md` | Questão em Aberto | Como monitorar que o worker está vivo | TRANSCRICAO | [09:11] Diego |
| RFC-QB-05 | `docs/RFC.md` | Questão em Aberto | Como fica a ordem por pedido quando um evento anterior está em retentativa | TRANSCRICAO | [09:13] Larissa |
| RFC-QB-06 | `docs/RFC.md` | Questão em Aberto | Quais respostas HTTP, além da falta de resposta, contam como falha | TRANSCRICAO | [09:42] Diego |
| RFC-QB-07 | `docs/RFC.md` | Questão em Aberto | Se a auditoria do reprocessamento fica só em log ou também persistida | TRANSCRICAO | [09:36] Sofia |
| RFC-QB-08 | `docs/RFC.md` | Questão em Aberto | Se o reprocessamento mantém o identificador original do evento | TRANSCRICAO | [09:18] Diego |
| RFC-QB-09 | `docs/RFC.md` | Questão em Aberto | Como assinar durante as 24 horas em que duas secrets são válidas | TRANSCRICAO | [09:21] Sofia |
| RFC-QB-10 | `docs/RFC.md` | Questão em Aberto | O timestamp de envio vai em header, mas fica fora da assinatura | TRANSCRICAO | [09:44] Diego |
| RFC-QB-11 | `docs/RFC.md` | Questão em Aberto | Como a secret é protegida em repouso | TRANSCRICAO | [09:21] Bruno |
| RFC-QB-12 | `docs/RFC.md` | Questão em Aberto | Se um evento enviado a dois webhooks do mesmo customer usa o mesmo identificador | TRANSCRICAO | [09:25] Diego |
| RFC-QB-13 | `docs/RFC.md` | Questão em Aberto | A validação de URL no schema gera o erro genérico de validação, e não o código próprio do módulo | TRANSCRICAO | [09:23] Sofia |
| RFC-QB-14 | `docs/RFC.md` | Questão em Aberto | A criação do pedido grava o status inicial fora da mudança de status; não está definido se isso gera evento | CODIGO | `src/modules/orders/order.service.ts` |
| RFC-RISCO-01 | `docs/RFC.md` | Risco | A transação de mudança de status, já pesada, fica mais lenta | TRANSCRICAO | [09:04] Bruno |
| RFC-RISCO-02 | `docs/RFC.md` | Risco | Um cliente não deduplica os eventos | TRANSCRICAO | [09:25] Sofia |
| RFC-RISCO-03 | `docs/RFC.md` | Risco | Uma secret vaza | TRANSCRICAO | [09:22] Diego |
| RFC-RISCO-04 | `docs/RFC.md` | Risco | O modelo de autorização é frouxo: qualquer usuário autenticado gerencia webhooks de qualquer customer, e o... | TRANSCRICAO | [09:37] Sofia |
| RFC-RISCO-05 | `docs/RFC.md` | Risco | O prazo pedido pela Atlas não é cumprido | TRANSCRICAO | [09:00] Marcos |
| FDD-REST-01 | `docs/FDD.md` | Restrição | Nada novo: reuso de `AppError`, Pino, error middleware, padrão de módulos, schemas Zod e códigos de erro | TRANSCRICAO | [09:29] Bruno |
| FDD-REST-02 | `docs/FDD.md` | Restrição | Mesmo banco e mesma stack; o worker abre um `PrismaClient` próprio, com o mesmo `DATABASE_URL` | TRANSCRICAO | [09:11] Diego |
| FDD-REST-03 | `docs/FDD.md` | Restrição | Um único worker; a ordem só vale por `order_id` e enquanto for single-worker | TRANSCRICAO | [09:13] Larissa |
| FDD-REST-04 | `docs/FDD.md` | Restrição | URL do webhook obrigatoriamente `https` | TRANSCRICAO | [09:23] Sofia |
| FDD-REST-05 | `docs/FDD.md` | Restrição | Payload de no máximo 64 KB, com erro se ultrapassar | TRANSCRICAO | [09:24] Larissa |
| FDD-OBJ-01 | `docs/FDD.md` | Objetivo | Atomicidade: se a transação principal commitou, o evento foi registrado; se deu rollback, o evento some junto | TRANSCRICAO | [09:06] Diego |
| FDD-OBJ-02 | `docs/FDD.md` | Objetivo | Latência: o worker lê a outbox a cada 2 s; a latência é de 2 s no pior caso, dentro da meta de 10 s | TRANSCRICAO | [09:10] Larissa |
| FDD-OBJ-03 | `docs/FDD.md` | Objetivo | Resiliência: 5 retentativas com backoff de 1 min, 5 min, 30 min, 2 h e 12 h, quase 15 h entre a primeira... | TRANSCRICAO | [09:17] Larissa |
| FDD-OBJ-04 | `docs/FDD.md` | Objetivo | Autenticidade: HMAC-SHA256 sobre o corpo, com secret por endpoint | TRANSCRICAO | [09:22] Sofia |
| FDD-OBJ-05 | `docs/FDD.md` | Objetivo | Deduplicação: UUID gerado quando o evento entra na outbox, único por evento, enviado em `X-Event-Id` e no... | TRANSCRICAO | [09:25] Diego |
| FDD-OBJ-06 | `docs/FDD.md` | Objetivo | Ordem: com um único worker, o processamento segue a ordem de `created_at` da outbox | TRANSCRICAO | [09:12] Diego |
| FDD-OBJ-07 | `docs/FDD.md` | Objetivo | Consistência: erros com o prefixo `WEBHOOK_`, no envelope do error middleware existente | TRANSCRICAO | [09:29] Larissa |
| FDD-PARAM-01 | `docs/FDD.md` | Parâmetro | Intervalo de polling | TRANSCRICAO | [09:10] Larissa |
| FDD-PARAM-02 | `docs/FDD.md` | Parâmetro | Tamanho do lote | TRANSCRICAO | [09:08] Diego |
| FDD-PARAM-03 | `docs/FDD.md` | Parâmetro | Timeout da chamada | TRANSCRICAO | [09:42] Diego |
| FDD-PARAM-04 | `docs/FDD.md` | Parâmetro | Retentativas e intervalos | TRANSCRICAO | [09:17] Larissa |
| FDD-PARAM-05 | `docs/FDD.md` | Parâmetro | Tamanho máximo do payload | TRANSCRICAO | [09:24] Larissa |
| FDD-PARAM-06 | `docs/FDD.md` | Parâmetro | Carência da secret antiga | TRANSCRICAO | [09:21] Sofia |
| FDD-PARAM-07 | `docs/FDD.md` | Parâmetro | Tipo do evento | TRANSCRICAO | [09:43] Diego |
| FDD-PARAM-08 | `docs/FDD.md` | Parâmetro | Itens do histórico | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-01 | `docs/FDD.md` | Contrato | Cadastrar webhook | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-02 | `docs/FDD.md` | Contrato | Listar webhooks de um customer | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-03 | `docs/FDD.md` | Contrato | Editar webhook | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-04 | `docs/FDD.md` | Contrato | Remover webhook | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-05 | `docs/FDD.md` | Contrato | Rotacionar secret | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-06 | `docs/FDD.md` | Contrato | Histórico de entregas | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-07 | `docs/FDD.md` | Contrato | Replay de evento da DLQ | TRANSCRICAO | [09:18] Diego |
| FDD-CONTRATO-08 | `docs/FDD.md` | Contrato | Envio ao cliente (saída do worker) | TRANSCRICAO | [09:43] Diego |
| FDD-CONTRATO-09 | `docs/FDD.md` | Contrato | Função de publicação (interna) | TRANSCRICAO | [09:41] Bruno |
| FDD-ERRO-01 | `docs/FDD.md` | Erro | `WEBHOOK_NOT_FOUND` | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-02 | `docs/FDD.md` | Erro | `WEBHOOK_INVALID_URL` | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-03 | `docs/FDD.md` | Erro | `WEBHOOK_SECRET_REQUIRED` | TRANSCRICAO | [09:28] Bruno |
| FDD-CA-01 | `docs/FDD.md` | Critério de Aceite | Mudar o status para um status que um webhook do customer quer ouvir insere o evento na outbox na mesma... | TRANSCRICAO | [09:40] Bruno |
| FDD-CA-02 | `docs/FDD.md` | Critério de Aceite | Se nenhum webhook do customer quer o status, nada é inserido | TRANSCRICAO | [09:34] Bruno |
| FDD-CA-03 | `docs/FDD.md` | Critério de Aceite | Se a inserção na outbox falhar, a mudança de status sofre rollback | TRANSCRICAO | [09:40] Bruno |
| FDD-CA-04 | `docs/FDD.md` | Critério de Aceite | O evento é entregue em menos de 10 s após a mudança de status | TRANSCRICAO | [09:02] Marcos |
| FDD-CA-05 | `docs/FDD.md` | Critério de Aceite | O envio traz `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type`, e a assinatura... | TRANSCRICAO | [09:44] Diego |
| FDD-CA-06 | `docs/FDD.md` | Critério de Aceite | Uma entrega que sempre falha é retentada 5 vezes, em 1 min, 5 min, 30 min, 2 h e 12 h, e termina na DLQ com... | TRANSCRICAO | [09:17] Larissa |
| FDD-CA-07 | `docs/FDD.md` | Critério de Aceite | Um cliente que não responde em 10 s conta como falha e vai para retry | TRANSCRICAO | [09:42] Diego |
| FDD-CA-08 | `docs/FDD.md` | Critério de Aceite | O replay exige role `ADMIN`, recoloca o evento na outbox como pendente e loga quem fez | TRANSCRICAO | [09:18] Diego |
| FDD-CA-09 | `docs/FDD.md` | Critério de Aceite | Uma URL `http` é recusada com erro de validação | TRANSCRICAO | [09:23] Sofia |
| FDD-CA-10 | `docs/FDD.md` | Critério de Aceite | Depois de uma rotação, a secret antiga fica válida por 24 h e depois deixa de valer | TRANSCRICAO | [09:21] Sofia |
| FDD-CA-11 | `docs/FDD.md` | Critério de Aceite | Um payload acima de 64 KB não é enviado | TRANSCRICAO | [09:23] Sofia |
| FDD-CA-12 | `docs/FDD.md` | Critério de Aceite | O histórico devolve os últimos 100 envios, com sucesso ou falha, payload, response e tempo de resposta | TRANSCRICAO | [09:34] Marcos |
| FDD-CA-13 | `docs/FDD.md` | Critério de Aceite | Com um único worker, os eventos de um pedido são entregues na ordem de `created_at` | TRANSCRICAO | [09:12] Diego |
| FDD-CA-14 | `docs/FDD.md` | Critério de Aceite | Os códigos de erro do módulo têm o prefixo `WEBHOOK_` | TRANSCRICAO | [09:29] Larissa |
| FDD-RISCO-01 | `docs/FDD.md` | Risco | A transação de mudança de status fica mais pesada | TRANSCRICAO | [09:04] Bruno |
| FDD-RISCO-02 | `docs/FDD.md` | Risco | O cliente recebe o mesmo evento duas vezes | TRANSCRICAO | [09:24] Diego |
| FDD-RISCO-03 | `docs/FDD.md` | Risco | Vazamento de secret | TRANSCRICAO | [09:22] Diego |
| FDD-RISCO-04 | `docs/FDD.md` | Risco | Autorização frouxa no cadastro e no replay | TRANSCRICAO | [09:37] Sofia |
| FDD-RISCO-05 | `docs/FDD.md` | Risco | Rajada de chamadas para um cliente | TRANSCRICAO | [09:38] Diego |
| FDD-INT-01 | `docs/FDD.md` | Integração | `src/modules/orders/order.service.ts` | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-INT-02 | `docs/FDD.md` | Integração | `src/app.ts` e `src/routes/index.ts` | CODIGO | `src/app.ts` |
| FDD-INT-03 | `docs/FDD.md` | Integração | `src/server.ts` e o novo `src/worker.ts` | CODIGO | `src/server.ts` |
| FDD-INT-04 | `docs/FDD.md` | Integração | `package.json` | CODIGO | `package.json` |
| FDD-INT-05 | `docs/FDD.md` | Integração | `src/config/database.ts` | CODIGO | `src/config/database.ts` |
| FDD-INT-06 | `docs/FDD.md` | Integração | `src/shared/errors/app-error.ts` e `src/shared/errors/http-errors.ts` | CODIGO | `src/shared/errors/app-error.ts` |
| FDD-INT-07 | `docs/FDD.md` | Integração | `src/middlewares/error.middleware.ts` | CODIGO | `src/middlewares/error.middleware.ts` |
| FDD-INT-08 | `docs/FDD.md` | Integração | `src/middlewares/auth.middleware.ts` e `src/modules/users/user.routes.ts` | CODIGO | `src/middlewares/auth.middleware.ts` |
| FDD-INT-09 | `docs/FDD.md` | Integração | `src/shared/logger/index.ts` | CODIGO | `src/shared/logger/index.ts` |
| FDD-INT-10 | `docs/FDD.md` | Integração | `prisma/schema.prisma` | CODIGO | `prisma/schema.prisma` |
| ADR-001 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Decisão | Outbox transacional no MySQL existente | TRANSCRICAO | [09:06] Diego |
| ADR-002 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Decisão | Worker em processo separado com polling | TRANSCRICAO | [09:11] Diego |
| ADR-003 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Decisão | Retry com backoff exponencial e DLQ em tabela separada | TRANSCRICAO | [09:17] Larissa |
| ADR-004 | `docs/adrs/ADR-004-autenticacao-hmac-sha256-com-secret-por-endpoint.md` | Decisão | Autenticação HMAC-SHA256 com secret por endpoint e rotação | TRANSCRICAO | [09:22] Sofia |
| ADR-005 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Decisão | Entrega at-least-once com X-Event-Id para deduplicação | TRANSCRICAO | [09:26] Larissa |
| ADR-006 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Reuso dos padrões existentes do projeto | TRANSCRICAO | [09:30] Larissa |
