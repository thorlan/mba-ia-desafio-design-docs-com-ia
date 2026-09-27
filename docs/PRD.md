### PRD: Order Management System, Sistema de Webhooks de Notificação de Pedidos

Versão: 1.0 (em revisão)
Data: Reunião técnica de quinta-feira, 09:00 (ver [`TRANSCRICAO.md`](../TRANSCRICAO.md))
Responsável: Diego (Engenheiro Sênior, time de Plataforma)

Documentos relacionados: [RFC](RFC.md), [FDD](FDD.md), [ADRs](adrs/). Convenção de fontes: `[hh:mm] Nome` aponta para a transcrição, e `caminho:linha` aponta para o código. O que não tem nenhuma das duas origens está marcado como **(hipótese)**.

---

### Resumo

Clientes B2B do OMS querem ser avisados quando o status dos seus pedidos muda, sem precisar consultar a API de tempos em tempos ([09:00] Marcos). Esta feature permite que cada cliente cadastre endpoints de webhook, escolha os status que quer receber e receba, em menos de 10 segundos, uma notificação assinada a cada mudança ([09:02] Marcos, [09:20] Sofia). A entrega é resiliente a indisponibilidades de até cerca de 15 horas, e os eventos que esgotam as tentativas ficam guardados para reprocessamento manual ([09:17] Diego, [09:18] Diego).

---

### Contexto e problema

Público-alvo
- Clientes B2B que integram com o OMS pela API; os primeiros são Atlas Comercial, MaxDistribuição e Nova Cargo ([09:00] Marcos).
- Desenvolvedores desses clientes, que implementam o recebimento e a verificação dos webhooks com a documentação do portal ([09:26] Marcos, [09:40] Marcos).
- Administradores da plataforma, que reprocessam eventos que falharam ([09:36] Sofia).

Cenários de uso chave
- Um pedido muda de status e o sistema do cliente é atualizado sozinho, sem polling ([09:00] Marcos).
- O cliente cadastra um webhook que só quer saber quando o pedido vira SHIPPED ou DELIVERED ([09:33] Marcos).
- O sistema do cliente fica duas horas fora do ar numa manutenção planejada e recebe os eventos quando volta ([09:16] Diego).
- Uma secret vaza num log do cliente, como já aconteceu, e ele pede uma nova sem interromper as entregas ([09:22] Diego, [09:21] Sofia).
- O cliente consulta as últimas entregas para entender uma falha ([09:34] Marcos).
- Um administrador reprocessa um evento que esgotou as tentativas ([09:18] Diego).

Onde essa feature será implantada
- No OMS existente: uma API REST em Node.js e TypeScript, monólito modular com um único processo HTTP e banco MySQL via Prisma (`src/server.ts:6`, `prisma/schema.prisma:5-9`). A feature entra como um módulo novo e um segundo processo, o worker de entregas ([09:27] Bruno, [09:11] Diego).

Problemas priorizados
- **Integração lenta e cara:** os clientes consultam a API de pedidos de tempos em tempos para descobrir mudanças, o que deixa a integração "lenta e cara pra eles" ([09:00] Marcos). Prioridade alta.
- **Risco de perder cliente:** a Atlas sinalizou que pode migrar para um concorrente se não tiver a funcionalidade até o fim do trimestre ([09:00] Marcos), e pediu a entrega para o fim de novembro ([09:45] Marcos). Prioridade alta.

---

### Objetivos e métricas

| Objetivo | Métrica | Meta |
| --- | --- | --- |
| PRD-OBJ-01: notificar o cliente em "tempo real" ([09:02] Marcos) | Tempo entre o commit da mudança de status e a primeira tentativa de entrega | Menos de 10 s ([09:02] Marcos); leitura pelo worker em até cerca de 2 s ([09:10] Larissa) |
| PRD-OBJ-02: nunca perder uma mudança de status ([09:40] Bruno) | Percentual de mudanças de status confirmadas, com webhook assinante, que têm evento gravado | 100% ([09:40] Bruno) |
| PRD-OBJ-03: absorver indisponibilidades do cliente sem perda ([09:16] Diego) | Janela coberta pelas retentativas antes de o evento ir para a DLQ | Cerca de 15 h ([09:17] Diego) |
| PRD-OBJ-04: entregar no prazo pedido pela Atlas ([09:45] Marcos) | Sprints até a feature estar em produção, com a revisão de segurança incluída | 3 sprints ([09:47] Larissa) |
| PRD-OBJ-05: substituir o polling dos clientes que pediram a feature ([09:00] Marcos) | Clientes com pelo menos um webhook ativo recebendo eventos | Os 3 clientes do pedido formal (hipótese: a reunião não fixou esta meta) |

---

### Escopo

Incluso
- Notificação automática a cada mudança de status de pedido, para os webhooks que assinam o novo status ([09:33] Marcos, [09:40] Bruno).
- Cadastro, listagem, edição e remoção de webhooks por customer ([09:31] Marcos, [09:33] Bruno).
- Rotação de secret com carência de 24 h ([09:21] Sofia).
- Assinatura HMAC-SHA256 e identificador único em cada envio ([09:22] Sofia, [09:25] Diego).
- Retentativas com backoff exponencial e DLQ ([09:17] Larissa).
- Histórico de entregas por webhook ([09:34] Marcos).
- Reprocessamento manual da DLQ por administradores, com auditoria ([09:36] Sofia).
- Documentação para os clientes no portal do desenvolvedor ([09:40] Marcos).

Fora de escopo
- **Webhooks de entrada:** os clientes "querem receber, não mandar" ([09:02] Marcos).
- **E-mail ao cliente em falhas seguidas:** "Email tá fora de escopo dessa fase. Talvez próxima fase, depois que a gente medir o impacto." ([09:37] Larissa).
- **Painel visual para o cliente:** "Não, agora não. Só endpoints. Painel é projeto separado do time de frontend." ([09:40] Larissa).
- **Rate limiting de envio:** adiado; fica como "observar e decidir depois" ([09:39] Larissa).
- **Vários workers em paralelo:** "isso é problema do futuro, não agora" ([09:13] Diego).
- **Arquivamento de eventos entregues:** "fora do escopo dessa feature" ([09:08] Diego).
- **Autorização refinada do cadastro de webhooks:** qualquer papel autenticado pode gerenciar webhooks "Por enquanto" ([09:37] Sofia).
- **Truncar payloads grandes:** descartado; acima do limite, o evento não é enviado ([09:23] Sofia, [09:24] Larissa).

---

### Requisitos funcionais

#### PRD-FR-01 Notificar mudanças de status
A cada mudança de status de um pedido, o sistema notifica os webhooks do customer que assinam o novo status ([09:00] Marcos, [09:34] Bruno).

**Fluxo principal**
- Um usuário muda o status de um pedido.
- Na mesma operação, o sistema registra um evento para cada webhook ativo do customer que assina o novo status ([09:06] Diego, [09:34] Bruno).
- O evento guarda um retrato do pedido naquele momento ([09:52] Larissa).
- Em até cerca de 2 s, o evento é enviado ao endpoint do cliente ([09:10] Larissa).

**Fluxos alternativos e exceções**
- Nenhum webhook assina o novo status: nenhum evento é registrado ([09:34] Bruno).
- Falha ao registrar o evento: a mudança de status é desfeita ([09:40] Bruno).

**Erros previstos**
- Falha de gravação do evento, que impede a mudança de status ([09:40] Bruno).

**Prioridade:** alta

---

#### PRD-FR-02 Cadastrar webhook
O cliente cadastra um endpoint de webhook para um customer, informando a URL e os status que quer receber; a plataforma gera a secret e a devolve na criação ([09:31] Marcos).

**Fluxo principal**
- O usuário autenticado envia o customer, a URL e a lista de status ([09:31] Marcos, [09:32] Larissa).
- O sistema valida a URL e gera uma secret única para o endpoint ([09:21] Sofia).
- O sistema devolve o webhook criado com a secret.

**Fluxos alternativos e exceções**
- O customer é informado no corpo ou no caminho da requisição, não vem do token ([09:32] Larissa).

**Erros previstos**
- URL que não é `https`: recusada com erro de validação ([09:23] Sofia).
- Customer inexistente.

**Prioridade:** alta

---

#### PRD-FR-03 Filtrar eventos por status
Cada webhook define a lista de status que quer receber, e só esses geram notificação ([09:33] Marcos).

**Fluxo principal**
- O cliente informa a lista de status no cadastro ou na edição, por exemplo SHIPPED e DELIVERED ([09:33] Marcos).
- Na mudança de status, só os webhooks que assinam o novo status recebem evento ([09:34] Bruno).

**Fluxos alternativos e exceções**
- O filtro é aplicado quando o evento é registrado, não no envio ([09:34] Bruno, [09:34] Diego).

**Erros previstos**
- Status fora da máquina de estados do pedido: erro de validação (`src/modules/orders/order.status.ts:3`).

**Prioridade:** alta

---

#### PRD-FR-04 Listar webhooks de um customer
O cliente lista os webhooks cadastrados de um customer ([09:33] Bruno).

**Fluxo principal**
- O usuário autenticado pede a lista informando o customer.
- O sistema devolve os webhooks, sem a secret.

**Fluxos alternativos e exceções**
- Customer sem webhooks: lista vazia.

**Erros previstos**
- Customer ausente ou inválido na requisição.

**Prioridade:** media

---

#### PRD-FR-05 Editar webhook
O cliente altera a URL, os status assinados ou o estado ativo de um webhook ([09:21] Bruno, [09:33] Bruno).

**Fluxo principal**
- O usuário envia os campos a alterar.
- O sistema valida e salva.

**Fluxos alternativos e exceções**
- Desativar o webhook interrompe o envio de novos eventos para ele ([09:21] Bruno).

**Erros previstos**
- Webhook inexistente ([09:28] Bruno).
- URL que não é `https` ([09:23] Sofia).

**Prioridade:** media

---

#### PRD-FR-06 Remover webhook
O cliente remove um webhook que não usa mais ([09:33] Bruno).

**Fluxo principal**
- O usuário pede a remoção.
- O sistema remove o webhook, e ele deixa de receber eventos.

**Fluxos alternativos e exceções**
- Destino dos eventos já registrados para o webhook removido: questão em aberto (RFC 5.2), com default proposto no FDD.

**Erros previstos**
- Webhook inexistente ([09:28] Bruno).

**Prioridade:** media

---

#### PRD-FR-07 Rotacionar secret
O cliente pede uma nova secret pela API; a anterior continua válida por 24 h em paralelo e depois é invalidada ([09:21] Sofia).

**Fluxo principal**
- O usuário pede a rotação de um webhook.
- O sistema gera a nova secret e a devolve.
- Durante 24 h, a secret anterior continua válida ([09:21] Sofia).

**Fluxos alternativos e exceções**
- Depois de 24 h, só a secret nova é válida ([09:21] Sofia).

**Erros previstos**
- Webhook inexistente.

**Prioridade:** alta

---

#### PRD-FR-08 Assinar cada envio
Todo envio carrega uma assinatura HMAC-SHA256 do corpo, calculada com a secret do endpoint, para o cliente verificar a origem e a integridade ([09:19] Sofia, [09:22] Sofia).

**Fluxo principal**
- O sistema calcula a assinatura do corpo com a secret do webhook.
- O sistema envia a assinatura num header ([09:20] Sofia).
- O cliente recalcula e compara.

**Fluxos alternativos e exceções**
- Durante a carência da rotação, a forma de assinar com duas secrets válidas é questão em aberto (RFC 5.2).

**Erros previstos**
- Assinatura que não confere, tratada do lado do cliente.

**Prioridade:** alta

---

#### PRD-FR-09 Identificar cada evento para deduplicação
Todo evento tem um identificador único, enviado em header e no corpo, para o cliente descartar repetições ([09:25] Diego, [09:43] Diego).

**Fluxo principal**
- O sistema gera um UUID quando registra o evento ([09:25] Diego, [09:51] Larissa).
- O identificador vai no header `X-Event-Id` e no corpo em todas as tentativas ([09:44] Diego).
- O cliente descarta eventos com identificador já processado ([09:25] Diego).

**Fluxos alternativos e exceções**
- Se o reprocessamento da DLQ mantém o identificador original: questão em aberto (RFC 5.2).

**Erros previstos**
- Nenhum do lado da plataforma; um cliente que não deduplica processa repetições (PRD-RISCO-02).

**Prioridade:** alta

---

#### PRD-FR-10 Retentar entregas com falha
Uma entrega com falha é retentada até 5 vezes, com intervalos de 1 min, 5 min, 30 min, 2 h e 12 h ([09:17] Diego, [09:17] Larissa).

**Fluxo principal**
- O envio falha, ou o cliente não responde em 10 s ([09:42] Diego).
- O sistema agenda a próxima tentativa conforme a progressão ([09:17] Diego).
- Na primeira resposta de sucesso, o evento é marcado como entregue.

**Fluxos alternativos e exceções**
- Depois do envio inicial e das 5 retentativas, o evento vai para a DLQ (PRD-FR-11).

**Erros previstos**
- Timeout de 10 s e respostas de erro do cliente ([09:42] Diego).

**Prioridade:** alta

---

#### PRD-FR-11 Guardar eventos esgotados na DLQ
Um evento que esgotou as tentativas vai para uma DLQ, com o payload, o motivo da falha e o momento ([09:18] Diego).

**Fluxo principal**
- A última tentativa falha.
- O sistema move o evento para a DLQ com o motivo e o momento ([09:18] Diego).

**Fluxos alternativos e exceções**
- Um evento acima de 64 KB não é enviado e também vai para a DLQ ([09:23] Sofia, [09:24] Larissa).

**Erros previstos**
- Tentativas esgotadas; payload acima do limite.

**Prioridade:** alta

---

#### PRD-FR-12 Reprocessar evento da DLQ
Um administrador devolve um evento da DLQ para a fila de envio, e a operação fica registrada com quem a fez ([09:18] Diego, [09:36] Sofia).

**Fluxo principal**
- Um administrador pede o reprocessamento de um evento da DLQ ([09:35] Diego).
- O sistema confere o papel de administrador ([09:36] Larissa).
- O evento volta para a fila como pendente ([09:18] Diego), e o sistema registra quem fez ([09:36] Sofia).

**Fluxos alternativos e exceções**
- O evento reprocessado segue as mesmas regras de retentativa.

**Erros previstos**
- Usuário sem papel de administrador: acesso negado ([09:36] Sofia).
- Evento da DLQ inexistente ou já reprocessado.

**Prioridade:** alta

---

#### PRD-FR-13 Consultar histórico de entregas
O cliente consulta as últimas 100 entregas de um webhook, com sucesso ou falha, payload, resposta e tempo de resposta ([09:34] Marcos).

**Fluxo principal**
- O usuário pede o histórico de um webhook.
- O sistema devolve as 100 entregas mais recentes ([09:34] Marcos).

**Fluxos alternativos e exceções**
- Webhook sem entregas: lista vazia.

**Erros previstos**
- Webhook inexistente.

**Prioridade:** media

---

### Requisitos não funcionais

Performance
- PRD-NFR-01: entrega em menos de 10 s após a mudança de status ([09:02] Marcos), com leitura pelo worker em até cerca de 2 s ([09:10] Larissa).
- PRD-NFR-02: timeout de 10 s por chamada ao cliente ([09:42] Diego).
- PRD-NFR-03: p95 menor que 150 ms nas rotas de cadastro, listagem e histórico (hipótese: default do curso; a reunião não deu número).
- PRD-NFR-04: o acréscimo na transação de mudança de status não tem teto definido; é questão em aberto (RFC 5.2).

Disponibilidade
- PRD-NFR-05: deploys e restarts da API não interrompem as entregas ([09:11] Diego).
- PRD-NFR-06: indisponibilidades do cliente de até cerca de 15 h não causam perda de eventos ([09:17] Diego).
- PRD-NFR-07: 99,9% de disponibilidade mensal para a API de webhooks (hipótese: default do curso para sistemas voltados ao cliente externo).

Segurança e autorização
- PRD-NFR-08: só URLs `https` ([09:23] Sofia).
- PRD-NFR-09: secret única por endpoint, rotacionável com carência de 24 h ([09:21] Sofia).
- PRD-NFR-10: todas as rotas exigem autenticação; o reprocessamento exige o papel de administrador e é auditado ([09:36] Sofia, [09:36] Larissa).
- PRD-NFR-11: a secret nunca aparece em log; hoje o logger mascara senhas e tokens, mas não secrets (`src/shared/logger/index.ts:4-11`).
- PRD-NFR-12: revisão de segurança do código antes do deploy ([09:46] Sofia).

Observabilidade
- PRD-NFR-13: logs estruturados com o logger Pino já existente ([09:29] Bruno).
- PRD-NFR-14: métricas de atraso, taxa de sucesso e tamanho da DLQ, e correlação por identificador do evento, sem biblioteca nova (detalhe na seção 7 do [FDD](FDD.md)).

Confiabilidade e integridade de dados
- PRD-NFR-15: mudança de status e registro do evento são atômicos ([09:06] Diego, [09:40] Bruno).
- PRD-NFR-16: entrega at-least-once ([09:24] Diego).
- PRD-NFR-17: o conteúdo do evento é o do momento da mudança, mesmo que o pedido mude depois ([09:52] Larissa).
- PRD-NFR-18: ordem de entrega por pedido, e só enquanto houver um único worker ([09:13] Larissa).

Compatibilidade e portabilidade
- PRD-NFR-19: API REST JSON sob o prefixo `/api/v1`, como o resto do sistema (`src/app.ts:67`).
- PRD-NFR-20: nenhuma biblioteca ou infraestrutura nova; o mesmo MySQL ([09:07] Diego, [09:29] Bruno).
- PRD-NFR-21: payload de no máximo 64 KB, com erro se ultrapassar ([09:24] Larissa).

Compliance
- PRD-NFR-22: trilha de auditoria de quem reprocessou cada evento ([09:36] Sofia). A reunião não levantou outro requisito de compliance.

Acessibilidade no frontend consumidor
- Não se aplica: a feature só expõe endpoints, e o painel visual está fora de escopo ([09:40] Larissa).

---

### Arquitetura e abordagem

Abordagem
- Comunicação assíncrona dentro do OMS existente: a mudança de status grava o evento numa outbox no MySQL, e um processo separado faz as entregas por polling ([09:06] Diego, [09:10] Larissa). Visão completa no [RFC](RFC.md).

Componentes
- Módulo de webhooks na API, no mesmo padrão dos demais módulos ([09:27] Bruno).
- Outbox, DLQ, configuração de webhooks e histórico de entregas no MySQL existente ([09:07] Diego).
- Worker de entregas, processo próprio com o mesmo banco e a mesma stack ([09:11] Diego).

Integrações
- Serviço de pedidos: a mudança de status passa a registrar o evento (`src/modules/orders/order.service.ts:126`).
- Endpoints HTTPS dos clientes, que recebem os envios assinados ([09:20] Sofia).
- Portal do desenvolvedor, onde os clientes aprendem a integrar ([09:40] Marcos).

### Decisões e trade-offs

#### Decisão: Outbox transacional no MySQL existente ([ADR-001](adrs/ADR-001-outbox-transacional-no-mysql.md))
- **Justificativa:** se a transação commita o evento existe, e se ela sofre rollback o evento some junto, sem infraestrutura nova ([09:06] Diego, [09:07] Diego).
- **Trade-off:** mais trabalho numa transação que já é pesada ([09:04] Bruno) e uma tabela que cresce até o arquivamento ([09:08] Diego).

#### Decisão: Worker em processo separado com polling ([ADR-002](adrs/ADR-002-worker-em-processo-separado-com-polling.md))
- **Justificativa:** o MySQL não avisa processos externos, e o worker não pode cair junto com a API ([09:09] Diego, [09:11] Diego).
- **Trade-off:** até cerca de 2 s de espera ([09:10] Larissa), dois processos para operar e uma instância só, sem escala horizontal ([09:13] Diego).

#### Decisão: Retry com backoff exponencial e DLQ ([ADR-003](adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md))
- **Justificativa:** cobre indisponibilidades de horas sem deixar eventos pendurados para sempre ([09:15] Diego, [09:16] Diego).
- **Trade-off:** um evento pode chegar horas depois, e o reprocessamento depende de um administrador ([09:18] Diego).

#### Decisão: HMAC-SHA256 com secret por endpoint e rotação ([ADR-004](adrs/ADR-004-autenticacao-hmac-sha256-com-secret-por-endpoint.md))
- **Justificativa:** padrão de mercado que qualquer cliente verifica, e um vazamento afeta um único endpoint ([09:20] Sofia, [09:21] Sofia).
- **Trade-off:** gerar, guardar e rotacionar secrets recuperáveis por endpoint.

#### Decisão: Entrega at-least-once com X-Event-Id ([ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md))
- **Justificativa:** nunca perder um evento, no padrão que os clientes já conhecem ([09:24] Diego, [09:25] Diego).
- **Trade-off:** "Isso joga responsabilidade pro cliente" de deduplicar ([09:25] Sofia).

#### Decisão: Reuso dos padrões existentes do projeto ([ADR-006](adrs/ADR-006-reuso-dos-padroes-do-projeto.md))
- **Justificativa:** erros e logs consistentes em toda a API, sem nada novo ([09:29] Bruno, [09:30] Larissa).
- **Trade-off:** herda limitações das classes de erro e dos middlewares atuais.

---

### Dependências

#### Organizacional: revisão de segurança
A Sofia precisa de pelo menos dois dias úteis para revisar o código de segurança, em especial o HMAC e a geração de secret, antes do deploy ([09:46] Sofia). A revisão está incluída nas três sprints ([09:47] Larissa).

#### Organizacional: documentação no portal do desenvolvedor
O Marcos documenta no portal como integrar via API ([09:40] Marcos), com destaque para a deduplicação pelo identificador do evento ([09:26] Marcos). Sem isso, os clientes não sabem verificar a assinatura nem deduplicar.

#### Externa: confirmação de prazo com a Atlas
O Marcos confirma com a Atlas o prazo de três sprints e atualiza os clientes ([09:47] Marcos, [09:49] Marcos).

#### Externa: adaptação dos sistemas dos clientes
Cada cliente precisa expor um endpoint HTTPS, verificar a assinatura e descartar eventos repetidos ([09:23] Sofia, [09:20] Sofia, [09:24] Diego).

#### Técnica: implantação do segundo processo
O worker roda como processo próprio, com a mesma configuração de ambiente da API ([09:11] Diego, `src/config/env.ts:3-10`), e precisa ser implantado e mantido junto com ela.

---

### Riscos e mitigação

#### PRD-RISCO-01: A entrega atrasa e a Atlas migra para um concorrente
- **Probabilidade:** media (hipótese: a estimativa de três sprints foi aceita, mas a reunião não comparou datas com o prazo de novembro)
- **Impacto:** perda de um cliente B2B ([09:00] Marcos).
- **Mitigação:**
  - Escopo enxuto: e-mail, painel e rate limiting fora desta fase ([09:48] Larissa).
  - Confirmação do prazo com a Atlas ([09:47] Marcos).
- **Plano de contingência:** entregar primeiro a notificação, o cadastro e o retry, e deixar o histórico de entregas para logo depois (hipótese).

#### PRD-RISCO-02: Um cliente não deduplica e processa o mesmo evento duas vezes
- **Probabilidade:** media (evidência: at-least-once entrega repetições por desenho, e a deduplicação fica com o cliente, [09:24] Diego, [09:25] Sofia)
- **Impacto:** efeito duplicado no sistema do cliente, como uma baixa repetida.
- **Mitigação:**
  - Documentação em destaque no portal ([09:26] Marcos).
  - Identificador estável em todas as tentativas ([09:25] Diego).
- **Plano de contingência:** o histórico de entregas mostra as tentativas de cada evento para o suporte ([09:34] Marcos).

#### PRD-RISCO-03: Uma secret vaza
- **Probabilidade:** media (evidência: "A gente já teve cliente que vazou secret em log de aplicação dele uma vez", [09:22] Diego)
- **Impacto:** terceiros conseguem forjar envios para aquele endpoint.
- **Mitigação:**
  - Secret por endpoint, que limita o alcance ([09:21] Sofia).
  - Mascaramento da secret nos logs da plataforma (`src/shared/logger/index.ts:4-11`).
  - Revisão de segurança antes do deploy ([09:46] Sofia).
- **Plano de contingência:** rotação da secret com carência de 24 h ([09:21] Sofia).

#### PRD-RISCO-04: Um usuário gerencia webhooks de outro customer ou reprocessa sem ser administrador legítimo
- **Probabilidade:** media (evidência: o cadastro fica aberto a qualquer papel autenticado, [09:37] Sofia; não há vínculo entre usuário e customer, `prisma/schema.prisma:25-54`; e o registro de usuário é público e aceita o papel de administrador, `src/modules/auth/auth.schemas.ts:7`)
- **Impacto:** alteração indevida de webhooks e reprocessamento não autorizado.
- **Mitigação:**
  - Reprocessamento restrito ao papel de administrador ([09:36] Larissa).
  - Auditoria de quem reprocessou ([09:36] Sofia).
  - Levar o ponto à revisão de segurança ([09:46] Sofia).
- **Plano de contingência:** endurecer a autorização numa fase seguinte, como previsto em [09:37] Sofia.

#### PRD-RISCO-05: A transação de mudança de status fica lenta demais
- **Probabilidade:** media (evidência: "A transação de mudança de status hoje já é pesada", [09:04] Bruno)
- **Impacto:** lentidão em toda mudança de status.
- **Mitigação:**
  - Só registrar evento quando algum webhook assina o status ([09:34] Bruno).
- **Plano de contingência:** medir antes e depois e definir o teto aceitável, que está em aberto (RFC 5.2).

#### PRD-RISCO-06: O worker para e as entregas atrasam para todos
- **Probabilidade:** baixa (hipótese)
- **Impacto:** a meta de 10 s deixa de ser cumprida enquanto o worker estiver parado.
- **Mitigação:**
  - Os eventos ficam guardados e são entregues quando o worker volta ([ADR-002](adrs/ADR-002-worker-em-processo-separado-com-polling.md)).
  - Alerta de atraso (seção 7 do [FDD](FDD.md)).
- **Plano de contingência:** reiniciar o worker.

#### PRD-RISCO-07: Um cliente recebe uma rajada de chamadas
- **Probabilidade:** baixa (evidência: o cenário foi levantado, mas o time decidiu que "A gente observa e implementa se virar problema", [09:39] Diego)
- **Impacto:** sobrecarga no endpoint do cliente, por exemplo 50 chamadas em um minuto ([09:38] Diego).
- **Mitigação:**
  - Observar o volume por cliente com as métricas de entrega.
- **Plano de contingência:** implementar rate limiting de saída, hoje fora de escopo ([09:39] Larissa).

#### PRD-RISCO-08: Um evento passa de 64 KB
- **Probabilidade:** baixa (evidência: "Nenhum evento nosso vai chegar perto disso", [09:24] Diego; o payload não leva os itens do pedido, [09:43] Diego)
- **Impacto:** o evento não é enviado.
- **Mitigação:**
  - Payload enxuto ([09:44] Bruno).
- **Plano de contingência:** o evento vai para a DLQ e pode ser analisado pelo administrador.

---

### Critérios de aceitação
Checklist objetivo que define se a feature está pronta. Os critérios técnicos detalhados estão na seção 9 do [FDD](FDD.md).

- PRD-CA-01: toda mudança de status para um status assinado gera exatamente um envio por webhook assinante, e mudanças desfeitas não geram envio.
- PRD-CA-02: o primeiro envio acontece em menos de 10 s após a mudança de status.
- PRD-CA-03: mudanças para status não assinados não geram envio.
- PRD-CA-04: o cliente consegue cadastrar, listar, editar e remover webhooks pela API, e URLs que não são `https` são recusadas.
- PRD-CA-05: todo envio traz assinatura verificável com a secret do endpoint e o identificador do evento.
- PRD-CA-06: depois de uma rotação, a secret anterior continua válida por 24 h e deixa de valer depois.
- PRD-CA-07: um endpoint fora do ar recebe o envio inicial e mais 5 retentativas nos intervalos combinados, e o evento termina na DLQ.
- PRD-CA-08: só administradores conseguem reprocessar um evento da DLQ, e cada reprocessamento registra quem fez.
- PRD-CA-09: o histórico devolve as 100 entregas mais recentes de um webhook, com resultado, resposta e tempo de resposta.
- PRD-CA-10: nenhuma secret aparece nos logs.
- PRD-CA-11: a revisão de segurança da Sofia foi feita antes do deploy.
- PRD-CA-12: a documentação de integração está publicada no portal do desenvolvedor.

---

### Testes e validação

Tipos de teste obrigatórios
- Testes de integração da API com Vitest e Supertest, no padrão atual (`tests/orders.test.ts`, `package.json:17`), cobrindo cadastro, rotação, histórico, reprocessamento e o registro do evento na mudança de status.
- Testes do worker com um endpoint simulado, cobrindo sucesso, timeout de 10 s, retentativas e DLQ, com relógio simulado para os intervalos.
- Testes unitários da assinatura HMAC e do cálculo do backoff.
- Teste de permissão: o reprocessamento responde acesso negado para quem não é administrador ([09:36] Sofia).
- Revisão de segurança manual do HMAC e da geração de secret ([09:46] Sofia).

Estratégia de validação
- Os testes automatizados cobrem os critérios técnicos do FDD antes do merge.
- Validação ponta a ponta com um endpoint de teste de um dos três clientes antes de liberar para todos (hipótese: a reunião não definiu a forma de homologação).
