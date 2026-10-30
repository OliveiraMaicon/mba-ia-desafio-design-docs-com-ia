# Potencial ADR: At least once com deduplicação pelo cliente

**Módulo:** WEBHOOKS
**Categoria:** contrato de entrega
**Prioridade:** consider — 75/150 (0 base + 25 impacto + 25 custo + 25 conhecimento)
**Identificado em:** 2026-10-30

## O que foi identificado

Diego descreve entrega at least once [09:24], UUID por evento no header `X-Event-Id` e deduplicação pelo cliente [09:25]; Larissa confirma [09:26] e retoma no resumo [09:48]. Diego contrasta com exactly once, que exigiria coordenação mais complexa [09:25].

## Por que documentar

- **Impacto:** contrato externo duradouro; consumidores precisam lidar com duplicatas.
- **Trade-off:** evita coordenação de exactly once, transferindo ao consumidor a responsabilidade de deduplicar.
- **Conhecimento:** replay e retry devem ser analisados sem prometer ausência de duplicatas. A DLQ após tentativas limitadas exige explicitar que entrega efetiva depende de recuperação: não é garantia incondicional de sucesso.

## Evidências

- `TRANSCRICAO.md`: Diego e Larissa [09:24–09:26]; Larissa [09:48]; Diego [09:15–09:18] descreve limite e DLQ. Marcos [09:26] compromete-se a destacar o contrato no portal.
- `prisma/schema.prisma` e `src/modules/orders/order.service.ts` mostram a persistência de pedidos atual, sem evento de webhook ou ID de entrega; não comprovam a garantia futura.
- `7ef4317` (2026-06-24, `init repository`) registra apenas a base; não demonstra evolução dessa garantia.

## Questões para o ADR

- O mesmo `event_id` será preservado entre todas as tentativas e replay? A reunião associa UUID ao evento, mas não detalha replay.
- Qual o escopo exato da garantia com DLQ e possível abandono operacional?
- Como orientar retenção de IDs e idempotência no consumidor sem inventar prazos?

## Candidatos relacionados

[Retry e DLQ](retry-backoff-dead-letter.md) · [Outbox](../../must-document/WEBHOOKS/outbox-transacional-mysql.md) · [HMAC](hmac-rotacao-segredo.md)
