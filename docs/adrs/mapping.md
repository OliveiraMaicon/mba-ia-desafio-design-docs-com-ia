# Mapeamento arquitetural para ADRs de webhooks

## Visão geral e fontes

OMS com API REST em TypeScript/Node.js (>=20), Express, Prisma e MySQL; dependências em `package.json` e datasource em `prisma/schema.prisma`. A fonte das decisões da feature é `TRANSCRICAO.md`, sobretudo o fechamento de Larissa [09:48] confirmado por Diego, Bruno e Sofia [09:49]. Código e histórico descrevem apenas o estado existente. `docs/FDD.md`, `docs/RFC.md` e `docs/PRD.md` são documentos de trabalho, não evidência independente de aprovação.

## Módulos e escopo

| ID | Responsabilidade, componentes e dependências | Escopo |
| --- | --- | --- |
| ORDERS | `src/modules/orders/`: controller, routes, schemas, service, repository e máquina de estados; usa Prisma e erros compartilhados. `order.service.ts` faz a transação de mudança de status e estoque. | Médio, 6 arquivos |
| WEBHOOKS | **Proposto, não implementado:** módulo `src/modules/webhooks` com configurações, entrega e processamento; integração com ORDERS, banco e HTTP de saída. Fonte: Bruno e Diego [09:27–09:28]. Escopo da identificação dos candidatos. | Ainda inexistente |
| AUTH | `src/modules/auth/` e `src/middlewares/auth.middleware.ts`: autenticação JWT e autorização por papel; relevante para CRUD e replay. | Pequeno, 4 arquivos do módulo e middleware |
| DOMAIN | `src/modules/customers/`, `products/` e `users/`: dados de cliente, produto e usuário, cada qual com controller, service, repository, routes e schemas. ORDERS depende de cliente e estoque. | Médio, 15 arquivos |
| PLATFORM | `src/app.ts`, `src/server.ts`, `src/routes/index.ts`, `src/config/`, `src/shared/`, `src/middlewares/`, `prisma/`: composição, servidor HTTP, configuração, persistência, logging, validação e erros. | Médio; diretórios compartilhados e migrations |
| TEST | `tests/`, `vitest.config.ts`: testes de integração da API, com banco; não demonstram webhooks existentes. | Separado; arquivos em `tests/` |

## Questões transversais e pontos de integração

- **Persistência:** `prisma/schema.prisma` usa MySQL; `OrderService.changeStatus` executa estoque, status e histórico na mesma transação (`src/modules/orders/order.service.ts:126–178`). Não há outbox nem DLQ no schema atual.
- **Execução:** `src/server.ts` é entrada da API; o worker separado e polling são decisões para a feature, não código existente ([09:09–09:11]).
- **Segurança:** JWT e `requireRole` existentes; assinatura HMAC por endpoint e rotação são decisões novas ([09:20–09:22], [09:35–09:36]).
- **Padrões:** módulos por domínio, Zod, Pino e tratamento compartilhado de erros são existentes (`src/modules/orders/`, `src/shared/`, `src/middlewares/`). A extensão a webhooks foi decidida na reunião [09:27–09:30].
- **Contratos e confiabilidade:** entrega outbound, retry, DLQ e deduplicação impactam clientes externos; implementação ainda ausente. Ordenação foi discutida para single-worker [09:12–09:13], mas retries podem criar reordenação; não assumir garantia implementada ou comprovada.
- **Histórico:** `git log --all` dos arquivos de origem, schema e transcrição aponta `7ef4317` (2026-06-24, `init repository`), sem mensagem que explique a origem das escolhas nem implementação da feature. Não estabelece a data da reunião nem a razão das decisões.

## Orientação para identificação

Analisar WEBHOOKS em conexão com ORDERS, AUTH e PLATFORM. Capturar apenas decisões da feature; não abrir candidatos genéricos para Express, MySQL ou Prisma preexistentes. Separar decisões confirmadas de propostas e perguntas em aberto, consolidar parâmetros dentro das estratégias e não promover detalhes do FDD a decisões. Consultar `docs/adrs/generated/` para duplicatas se existir; nesta análise não havia ADR formal nesse diretório nem em `docs/adrs/`.
