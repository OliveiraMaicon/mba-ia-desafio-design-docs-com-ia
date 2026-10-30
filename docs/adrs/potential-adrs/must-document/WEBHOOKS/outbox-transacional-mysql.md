# Potencial ADR: Outbox transacional no MySQL

**Módulo:** WEBHOOKS / ORDERS
**Categoria:** infraestrutura crítica
**Prioridade:** must-document — 120/150 (75 base + 20 impacto + 15 custo + 10 conhecimento)
**Identificado em:** 2026-10-30

## O que foi identificado

Diego explica [09:06] registrar evento na mesma transação SQL que muda pedido e histórico; Larissa decide outbox no MySQL [09:08] e confirma no resumo [09:48]. Bruno e Diego reforçam rollback conjunto [09:40–09:41]. Larissa e Diego escolhem payload snapshot na inserção [09:52]. Filtragem por status é combinada antes da inserção [09:33–09:34].

## Por que documentar

- **Impacto:** conecta mudança de pedido, persistência e entregas externas; falha de inserção deve reverter a mudança.
- **Trade-off:** acoplamento ao banco e crescimento da tabela em troca de atomicidade, sem chamada HTTP durante a transação.
- **Conhecimento e futuro:** engenheiros precisam preservar a fronteira transacional; arquivamento após aproximadamente 30 dias foi mencionado como fora do escopo [09:08].

## Evidências

- `TRANSCRICAO.md`: Bruno e Larissa [09:04] rejeitam envio síncrono; Larissa e Diego [09:07] discutem Redis e custo de infraestrutura; Diego [09:08] propõe índices e batches. Não há decisão de broker.
- `src/modules/orders/order.service.ts:126–178`: transação Prisma existente; estoque, `tx.order.update` e `tx.orderStatusHistory.create` ocorrem dentro dela. Não existe inserção em outbox.
- `prisma/schema.prisma:5–9,74–97,116–131`: MySQL, pedidos e histórico existentes, sem modelos de webhook.
- A base desses arquivos aparece no commit `7ef4317` de 2026-06-24 (`init repository`); o commit não prova a data ou motivação da escolha de outbox.

## Questões para o ADR

- Como representar os eventos elegíveis e manter o snapshot consistente com a transação?
- Como lidar com crescimento e arquivamento, sem antecipar a implementação fora de escopo?
- Como demonstrar atomicidade em testes e qual o efeito de falha da outbox sobre o pedido?

## Candidatos relacionados

[Worker separado](../../consider/WEBHOOKS/worker-separado-polling.md) · [Retry e DLQ](../../consider/WEBHOOKS/retry-backoff-dead-letter.md) · [At least once](../../consider/WEBHOOKS/at-least-once-deduplicacao.md)
