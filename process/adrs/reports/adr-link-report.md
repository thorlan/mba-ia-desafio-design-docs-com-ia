# Relatório de relações entre ADRs

Gerado com o prompt [`04-adr-link-adaptado.md`](../../prompts/04-adr-link-adaptado.md) sobre as ADR-001 a ADR-006.

## Resumo

- ADRs processadas: 6, todas do módulo WEBHOOKS.
- Supersessão e emenda: nenhuma, porque todas as decisões são da mesma reunião.
- Depende de / Usada por: 6 relações.
- Relacionada a: 5 pares recíprocos.
- ADR fundacional: ADR-006 (reuso dos padrões), que não aparece como alvo de "Depende de".

## Depende de

| ADR | Depende de | Evidência |
| --- | --- | --- |
| ADR-002 | ADR-001 | O Contexto parte dos eventos gravados na outbox, que o worker lê. |
| ADR-003 | ADR-001 | O reprocessamento devolve o evento à outbox como pendente. |
| ADR-003 | ADR-002 | Quem faz as retentativas é o worker, e o timeout é o da chamada dele. |
| ADR-005 | ADR-001 | O identificador é gerado na gravação na outbox, e o retrato gravado garante o mesmo conteúdo no reenvio. |
| ADR-005 | ADR-002 | O Contexto cita o worker como uma das causas de entrega repetida. |
| ADR-005 | ADR-003 | O Contexto cita as retentativas como causa de entrega repetida, e as Negativas tratam o reprocessamento da DLQ. |

## Relacionada a

| Par | Evidência |
| --- | --- |
| ADR-001 e ADR-006 | A integração com pedidos, escolhida na ADR-006, é o ponto em que a ADR-001 grava o evento. |
| ADR-002 e ADR-004 | O worker usa a secret para assinar cada envio, e por isso ela precisa ser recuperável. |
| ADR-002 e ADR-006 | A ADR-006 registra que o error middleware não cobre o worker e que os logs dele saem com o nome de serviço da API. |
| ADR-003 e ADR-006 | O reprocessamento reaproveita o controle de acesso por papel existente. |
| ADR-004 e ADR-005 | Os dois headers da entrega (`X-Signature` e `X-Event-Id`) formam o contrato com o cliente. |

## Descartes

- **ADR-004 e ADR-006:** estavam relacionadas na versão anterior. Com o limite de 3 por tipo, a ADR-006 ficou com as relações de evidência mais forte (001, 002 e 003). A ADR-004 cita o logger só como constatação, sem depender da decisão de reuso.
- **ADR-001 e ADR-004:** sem evidência no texto das duas.
