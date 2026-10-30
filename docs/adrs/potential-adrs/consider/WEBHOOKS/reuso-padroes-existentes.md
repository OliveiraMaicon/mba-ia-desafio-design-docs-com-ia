# Potencial ADR: Reuso dos padrões existentes na feature

**Módulo:** WEBHOOKS / PLATFORM
**Categoria:** organização arquitetural
**Prioridade:** consider — 45/150 (0 base + 20 impacto + 10 custo + 15 conhecimento); exceção ao limiar, conforme pedido
**Identificado em:** 2026-10-30

## O que foi identificado

Bruno propõe módulo de webhook com controller, service, repository, routes e schemas [09:27]; ele e Diego discutem entrada separada e processamento no módulo [09:28]. Larissa decide reusar AppError, Pino, middleware de erros, Zod e códigos com prefixo `WEBHOOK_` [09:29–09:30]. É decisão de aplicação dos padrões à feature, não decisão original pela stack.

## Por que documentar

- **Impacto:** afeta fronteiras do módulo, erros, validação e observabilidade.
- **Trade-off:** consistência e menor variedade tecnológica em troca de aderência aos padrões existentes.
- **Conhecimento:** torna explícita a separação entre worker e API e quais convenções devem ser preservadas.

## Evidências

- `TRANSCRICAO.md`: Bruno e Larissa [09:27–09:30]; o resumo de Larissa [09:48] confirma os padrões.
- `src/modules/orders/` contém as camadas citadas; `src/shared/errors/app-error.ts`, `src/shared/logger/index.ts` e `src/middlewares/error.middleware.ts` exemplificam reuso existente; `src/modules/orders/order.schemas.ts` utiliza schemas. Não existe ainda `src/modules/webhooks/`.
- `7ef4317` (2026-06-24, `init repository`) contém a base; seu título não explica a adoção inicial dos padrões.

## Questões para o ADR

- Como estruturar o módulo e preservar separação de processo e composição?
- Quais erros de webhook devem seguir as convenções existentes sem inventar um catálogo fechado?
- Como observar falhas do worker, que não percorrem necessariamente o middleware HTTP?

## Candidatos relacionados

[Worker](worker-separado-polling.md) · [HMAC](hmac-rotacao-segredo.md) · [Retry e DLQ](retry-backoff-dead-letter.md)
