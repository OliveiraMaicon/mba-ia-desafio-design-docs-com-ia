# ADR-006: Reuso dos padrões existentes em webhooks

- **Status:** Aceito
- **Decisora:** Larissa

## Contexto e problema

A funcionalidade de webhooks precisará de uma interface HTTP e de processamento assíncrono, mas ainda não existe módulo de webhooks no código. Como organizá-la e tratar validação, erros e logs sem criar convenções incompatíveis com o OMS?

O módulo de pedidos já separa controller, service, repository, routes e schemas em `src/modules/orders/`. A aplicação compõe dependências em `src/app.ts`, monta rotas em `src/routes/index.ts` e usa `AppError`, validação Zod e logger Pino compartilhados. Bruno propôs seguir essa estrutura para webhooks; Larissa confirmou o reuso dos padrões e o prefixo `WEBHOOK_` para erros do módulo ([transcrição](../../TRANSCRICAO.md), Bruno e Larissa, 09:27–09:30).

## Direcionadores da decisão

- Consistência com os módulos, erros e observabilidade já conhecidos pela equipe.
- Menor variedade de tecnologias e convenções para manter.
- Separação entre a entrada HTTP e o processo de trabalho assíncrono.

## Opções consideradas

1. **Reusar os padrões existentes:** organizar webhooks como módulo do OMS e usar as ferramentas compartilhadas.
2. **Criar estrutura e ferramentas próprias para webhooks:** permitir convenções especializadas, ao custo de duplicação e maior variedade operacional. Esta é uma alternativa plausível identificada para comparação, não uma proposta registrada na reunião.
3. **Acrescentar a lógica aos módulos existentes, sem módulo próprio:** evitar uma nova estrutura, mas misturar responsabilidades de pedidos e entrega. Também é uma alternativa plausível identificada, não uma proposta registrada na reunião.

## Decisão

Adotar a primeira opção: propor `src/modules/webhooks/` com controller, service, repository, routes e schemas, seguindo o padrão dos demais domínios. Reusar `AppError`, o middleware centralizado de erros HTTP, schemas Zod e Pino; erros específicos do módulo seguem o prefixo `WEBHOOK_`. Os nomes citados por Bruno, como `WEBHOOK_NOT_FOUND` e `WEBHOOK_INVALID_URL`, são exemplos, não um catálogo fechado ([transcrição](../../TRANSCRICAO.md), Bruno e Larissa, 09:28–09:30).

O worker terá entrada de processo separada, proposta como `src/worker.ts`, com lógica de processamento no módulo de webhooks. Por ser outro processo, terá sua própria instância de PrismaClient, conectada ao mesmo banco pela mesma `DATABASE_URL` ([transcrição](../../TRANSCRICAO.md), Bruno e Diego, 09:28–09:30). Esses caminhos de webhooks e do worker são propostas, não arquivos existentes.

O prefixo não implica que todo erro produzido pela infraestrutura compartilhada já tenha esse código: `src/middlewares/validate.middleware.ts` converte erros Zod em `ValidationError`, e o tratamento existente retorna `VALIDATION_ERROR`; `src/middlewares/error.middleware.ts` também trata diretamente `ZodError` com esse código. A eventual necessidade de traduzir uma falha de validação específica para `WEBHOOK_` e seu contrato permanecem em aberto; não se pressupõe que o middleware atual faça essa tradução.

## Consequências

### Positivas

- A feature segue fronteiras e convenções familiares, reduzindo duplicação e variedade tecnológica.
- A API pode aproveitar o tratamento centralizado de erros e a validação já utilizados pelas rotas existentes.
- A entrada separada permite executar o worker independentemente da API, mantendo sua lógica no domínio de webhooks.

### Negativas e pontos de atenção

- A adesão aos padrões existentes limita escolhas especializadas e pode exigir adaptação quando não cobrirem necessidades da feature.
- O middleware HTTP não intercepta automaticamente falhas do worker: seu tratamento e registro precisam ser considerados no próprio processo. A forma específica de observação dessas falhas não foi fechada na reunião.
- O comportamento genérico de validação e os códigos específicos `WEBHOOK_` devem ser conciliados explicitamente, sem prometer que todas as respostas de erro já terão o prefixo.
- Cada processo terá seu próprio cliente Prisma; compartilhar o banco não significa compartilhar a instância ou o pool em memória.

## Relações

- [ADR-002 — Worker separado com polling da outbox](ADR-002-worker-separado-polling.md): define o processo separado e o polling, cuja organização é detalhada aqui.
- [ADR-004 — HMAC e rotação de segredos](ADR-004-hmac-e-rotacao-de-segredos.md): define HTTPS para URLs cadastradas, relacionado à validação do módulo de webhooks.

## Fontes

- [Transcrição](../../TRANSCRICAO.md): Bruno, Diego e Larissa, 09:27–09:30; Larissa, 09:48, confirma o reuso.
- [Rotas de pedidos](../../src/modules/orders/order.routes.ts) e [schemas de pedidos](../../src/modules/orders/order.schemas.ts): exemplos de organização e validação.
- [Composição da aplicação](../../src/app.ts) e [rotas da API](../../src/routes/index.ts): integração dos módulos e middleware HTTP.
- [AppError](../../src/shared/errors/app-error.ts), [middleware de validação](../../src/middlewares/validate.middleware.ts), [middleware de erros](../../src/middlewares/error.middleware.ts) e [logger](../../src/shared/logger/index.ts): infraestrutura compartilhada existente.
