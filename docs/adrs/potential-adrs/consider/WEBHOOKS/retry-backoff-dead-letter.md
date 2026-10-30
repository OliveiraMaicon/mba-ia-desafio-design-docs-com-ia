# Potencial ADR: Retry com backoff e dead letter queue

**Módulo:** WEBHOOKS
**Categoria:** resiliência
**Prioridade:** consider — 50/150 (0 base + 20 impacto + 15 custo + 15 conhecimento); exceção ao limiar, conforme pedido
**Identificado em:** 2026-10-30

## O que foi identificado

Diego propõe backoff e limite de tentativas com DLQ [09:15]; Larissa decide cinco tentativas e intervalos 1m/5m/30m/2h/12h [09:17]. Diego propõe tabela separada e replay manual via endpoint admin [09:18]; Larissa registra [09:19], Sofia exige ADMIN e auditoria [09:36]. Larissa confirma a estratégia no resumo [09:48].

## Por que documentar

- **Impacto:** define comportamento quando clientes ficam indisponíveis e o caminho operacional de recuperação.
- **Trade-off:** limite evita retry infinito, mas deixa eventos não entregues para intervenção; tabela separada facilita investigação e adiciona manutenção.
- **Conhecimento:** a equipe deve entender falhas, auditoria e consequências do replay; tempo de timeout HTTP de 10 s foi proposto por Diego e aceito por Sofia [09:42].

## Evidências

- `TRANSCRICAO.md`: Diego e Bruno discutem três, cinco e retry indefinido [09:15–09:17]; Diego contrapõe tabela separada a marcação na outbox [09:17–09:18].
- `prisma/schema.prisma` não contém outbox ou DLQ; `src/middlewares/auth.middleware.ts` contém o padrão de autorização a reaproveitar. Nenhum processamento de entrega está presente em `src/`.
- O histórico dos arquivos de base disponíveis é `7ef4317` (2026-06-24, `init repository`), não uma implementação ou explicação da estratégia.

## Questões para o ADR

- Cinco tentativas incluem a primeira chamada ou são cinco retentativas? Como aplicar os cinco intervalos sem contradizer a contagem?
- Quais respostas e falhas entram em retry, como registrar motivo e como preservar auditoria no replay?
- Como retry e replay interagem com event ID e ordenação?

## Candidatos relacionados

[Worker](worker-separado-polling.md) · [At least once](at-least-once-deduplicacao.md) · [Outbox](../../must-document/WEBHOOKS/outbox-transacional-mysql.md)
