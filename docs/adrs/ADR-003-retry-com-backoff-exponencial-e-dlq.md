# ADR-003: Retry com backoff exponencial e DLQ em tabela separada

**Status:** Aceito
**Data:** Reunião técnica de quinta-feira, 09:00 (ver `TRANSCRICAO.md`)
**Depende de:**
- [ADR-001: Outbox transacional no MySQL existente](./ADR-001-outbox-transacional-no-mysql.md)
- [ADR-002: Worker em processo separado com polling](./ADR-002-worker-em-processo-separado-com-polling.md)

**Usada por:** [ADR-005: Entrega at-least-once com X-Event-Id para deduplicação](./ADR-005-entrega-at-least-once-com-x-event-id.md)
**Relacionada a:** [ADR-006: Reuso dos padrões existentes do projeto](./ADR-006-reuso-dos-padroes-do-projeto.md)

## Contexto e Problema

O cliente pode estar offline ([09:14] Larissa). Já houve cliente com duas horas de indisponibilidade numa manutenção planejada ([09:16] Diego).

Desistir cedo demais não cobre indisponibilidades como essa ([09:16] Diego). Insistir para sempre deixa o evento pendurado se o cliente sumiu ([09:15] Diego).

A reunião também tratou do evento que esgota as tentativas: onde ele fica ([09:17] Larissa), quem o reprocessa ([09:18] Bruno) e como isso é auditado ([09:36] Sofia).

## Fatores de Decisão

- Cobrir indisponibilidades de horas, como a manutenção de duas horas já vista ([09:16] Diego).
- Não manter eventos pendurados para sempre ([09:15] Diego).
- Manter enxuta a leitura de pendentes feita pelo worker ([09:18] Diego).
- Guardar evidência dos eventos que falharam, para debug e reprocessamento ([09:18] Diego).
- Restringir o reprocessamento a administradores, com registro de quem o fez ([09:36] Sofia).

## Alternativas Consideradas

1. **Backoff exponencial com teto de tentativas e DLQ em tabela separada.**
2. **Retry indefinido com backoff.**
3. **Teto de 3 tentativas.**

## Decisão

Alternativa escolhida: **backoff exponencial com teto de tentativas e DLQ em tabela separada** ([09:17] Larissa), porque cobre indisponibilidades de horas ([09:16] Diego) sem deixar eventos pendurados para sempre ([09:15] Diego). Depois do envio inicial, uma entrega com falha é retentada em até **5 novas tentativas**, com intervalos de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas ([09:17] Diego). No máximo, são **6 chamadas HTTP por evento**, distribuídas ao longo de 14h36 entre a primeira falha e a última tentativa, as "quase 15 horas" citadas em [09:17] Diego. Essa é a interpretação adotada neste pacote de documentos, por ser a única que fecha com a soma dos intervalos e com o exemplo de "três vezes em 30 minutos" ([09:16] Diego). Uma chamada sem resposta em 10 segundos conta como falha ([09:42] Diego).

Esgotadas as tentativas, o evento vai para uma **DLQ em tabela própria**, em vez de ficar marcado como falho na outbox ([09:17] Larissa), porque isso deixa mais limpa a leitura da outbox principal e guarda evidência para debug e reprocessamento ([09:18] Diego). O reprocessamento é **manual**, por uma operação administrativa que devolve o evento à outbox como pendente ([09:18] Diego). Ela exige o papel de administrador, reaproveitando o controle de acesso por papel que já existe, e registra quem a executou ([09:36] Sofia, [09:36] Larissa).

## Prós e Contras das Alternativas

### Backoff exponencial com teto e DLQ separada
- Pró: cobre quase 15 horas entre a primeira falha e a última tentativa ([09:17] Diego).
- Pró: a DLQ separada deixa mais limpa a leitura da outbox principal e guarda evidência para debug e reprocessamento ([09:18] Diego).
- Contra: a última retentativa acontece 12 horas depois da anterior ([09:17] Diego).
- Contra: o reprocessamento é manual e exige administrador ([09:18] Diego, [09:36] Sofia).

### Retry indefinido com backoff
- Contra: eventos ficam pendurados para sempre quando o cliente some ([09:15] Diego).

### Teto de 3 tentativas
- Pró: mais agressivo, desiste do evento mais cedo ([09:16] Bruno).
- Contra: "3 é pouco": a plataforma retentaria "três vezes em 30 minutos" e desistiria ([09:16] Diego).
- Contra: não cobre a manutenção planejada de duas horas que um cliente já teve ([09:16] Diego).

## Consequências

**Positivas.** Indisponibilidades de até quase 15 horas, como a manutenção de duas horas já vista, são cobertas pelas retentativas ([09:16] Diego, [09:17] Diego). Os eventos esgotados ficam na DLQ como evidência para debug e reprocessamento ([09:18] Diego). Acima de 15 horas de indisponibilidade o problema é do cliente ([09:17] Marcos).

**Negativas.** Um evento pode chegar até quase 15 horas depois da primeira falha ([09:17] Diego). O cliente não é avisado de forma proativa quando suas entregas falham, porque o aviso por e-mail ficou fora do escopo desta fase ([09:37] Larissa). A reunião não definiu três pontos, que ficam como questões em aberto no RFC: quais respostas HTTP, além da falta de resposta, contam como falha; se o reprocessamento mantém o identificador original do evento; e se o registro de auditoria do reprocessamento é só em log ou também persistido.

**Trade-off explícito:** aceitamos entregas até quase 15 horas depois ([09:17] Diego) e reprocessamento manual ([09:18] Diego) em troca de cobrir indisponibilidades de horas ([09:16] Diego) sem deixar eventos pendurados para sempre ([09:15] Diego).

## Referências

- `src/middlewares/auth.middleware.ts:49` (controle de acesso por papel, reaproveitado no reprocessamento)
- `src/modules/users/user.routes.ts:15` (uso atual da restrição ao papel de administrador)
- `src/shared/logger/index.ts:13` (logger estruturado, destino do registro de auditoria)
- `prisma/schema.prisma:116` (histórico de status: padrão atual de tabela de registro com UUID e índices)
- Transcrição: [09:14] Larissa, [09:15] Diego, [09:16] Diego, [09:16] Bruno, [09:17] Larissa, [09:17] Diego, [09:17] Marcos, [09:18] Bruno, [09:18] Diego, [09:36] Sofia, [09:36] Larissa, [09:37] Larissa, [09:42] Diego
