# Tracker de rastreabilidade

Os IDs abaixo cobrem os itens do PRD, da RFC, do FDD e dos ADRs; hipóteses, avaliações e questões em aberto estão identificadas nos documentos.

Uma linha pode agrupar itens relacionados ou apontar várias fontes. Em linhas de **hipótese** ou **questão em aberto**, a localização mostra os fatos que motivam a análise, não uma aprovação da proposta nem evidência de que a questão foi discutida explicitamente. Os códigos, formatos de resposta, exemplos de payload e métricas propostos no FDD precisam de revisão; não foram decididos na reunião. Os IDs já presentes nos documentos são preservados; IDs adicionais identificam itens por seção ou contrato.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-CTX-01 | docs/PRD.md | Problema | Clientes B2B consultam pedidos repetidamente; risco de perda da Atlas | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-02 | docs/PRD.md | Contexto | Aplicação sem mecanismo de webhooks ou notificações externas | CODIGO | src/app.ts; prisma/schema.prisma |
| PRD-MET-01 | docs/PRD.md | Objetivo | Clientes aceitam tempo abaixo de 10 segundos; polling a cada 2 s | TRANSCRICAO | [09:02] Marcos; [09:09] Diego |
| PRD-PLAN-01 | docs/PRD.md | Planejamento | Pedido para novembro, estimativa de três sprints e revisão de segurança | TRANSCRICAO | [09:45] Marcos; [09:46] Larissa; [09:46] Sofia; [09:47] Marcos |
| PRD-RF-01 | docs/PRD.md | Requisito funcional | Cadastro, secret gerada, JWT e customer explícito | TRANSCRICAO | [09:31] Marcos; [09:32] Larissa |
| PRD-RF-02 | docs/PRD.md | Requisito funcional | Editar, remover e listar webhooks com seleção de status | TRANSCRICAO | [09:33] Bruno; [09:33] Marcos; [09:37] Sofia |
| PRD-RF-03 | docs/PRD.md | Requisito funcional | Filtrar status antes da inserção na outbox | TRANSCRICAO | [09:33] Marcos; [09:34] Bruno |
| PRD-RF-04 | docs/PRD.md | Requisito funcional | Outbox atômica com mudança de status e snapshot do payload | TRANSCRICAO | [09:40] Bruno; [09:52] Larissa |
| PRD-RF-05 | docs/PRD.md | Requisito funcional | Entrega at-least-once e deduplicação por ID de evento | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Larissa |
| PRD-RF-06 | docs/PRD.md | Requisito funcional | Cinco tentativas, intervalos e DLQ separada | TRANSCRICAO | [09:17] Larissa; [09:18] Diego; [09:48] Larissa |
| PRD-RF-07 | docs/PRD.md | Requisito funcional | Rotação de secret com sobreposição de 24 horas | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia |
| PRD-RF-08 | docs/PRD.md | Requisito funcional | Histórico dos últimos 100 envios | TRANSCRICAO | [09:34] Marcos |
| PRD-RF-09 | docs/PRD.md | Requisito funcional | Replay manual de DLQ por ADMIN, com auditoria | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| PRD-RF-10 | docs/PRD.md | Requisito funcional | Documentar integração e deduplicação para clientes | TRANSCRICAO | [09:26] Marcos; [09:40] Marcos |
| PRD-SCOPE-01 | docs/PRD.md | Restrição | Webhooks apenas de saída | TRANSCRICAO | [09:02] Marcos |
| PRD-SCOPE-02 | docs/PRD.md | Exclusão | Email de falha fora desta fase | TRANSCRICAO | [09:37] Larissa |
| PRD-SCOPE-03 | docs/PRD.md | Exclusão | Dashboard fora desta fase | TRANSCRICAO | [09:40] Larissa |
| PRD-SCOPE-04 | docs/PRD.md | Questão em aberto | Rate limiting de saída será observado | TRANSCRICAO | [09:39] Diego; [09:39] Larissa |
| PRD-SCOPE-05 | docs/PRD.md | Exclusão | Arquivamento e múltiplos workers ficam para depois | TRANSCRICAO | [09:08] Diego; [09:13] Diego; [09:13] Larissa |
| PRD-DEC-01 | docs/PRD.md | Decisão | Outbox MySQL transacional e worker separado com polling | TRANSCRICAO | [09:04] Bruno; [09:06] Diego; [09:07] Diego; [09:09] Diego; [09:11] Diego |
| PRD-DEC-02 | docs/PRD.md | Decisão | Um worker e ordenação pretendida por pedido, não global | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| PRD-NFR-01 | docs/PRD.md | Segurança | HTTPS, HMAC, secrets únicas, autenticação e autorização por role | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:23] Sofia; [09:36] Sofia; [09:37] Sofia |
| PRD-NFR-02 | docs/PRD.md | Contrato | Campos do JSON e headers de entrega | TRANSCRICAO | [09:43] Diego; [09:44] Diego; [09:44] Sofia |
| PRD-NFR-03 | docs/PRD.md | Requisito não funcional | Limite de 64 KB e timeout HTTP de 10 s | TRANSCRICAO | [09:24] Larissa; [09:42] Diego |
| PRD-NFR-04 | docs/PRD.md | Restrição | Reuso dos padrões do projeto e PrismaClient por processo | TRANSCRICAO | [09:28] Bruno; [09:29] Bruno; [09:30] Bruno; [09:30] Larissa |
| PRD-DEP-01 | docs/PRD.md | Dependência | MySQL, Prisma e transação de mudança de status | CODIGO | prisma/schema.prisma; src/modules/orders/order.service.ts |
| PRD-DEP-02 | docs/PRD.md | Risco | JWT não associa usuário a customer; autorização exige definição | CODIGO | src/middlewares/auth.middleware.ts; prisma/schema.prisma |
| PRD-RISK-01 | docs/PRD.md | Risco | Indisponibilidade ou lentidão do endpoint | TRANSCRICAO | [09:14] Larissa; [09:15] Diego; [09:42] Diego |
| PRD-RISK-02 | docs/PRD.md | Risco | Vazamento de secret ou adulteração | TRANSCRICAO | [09:19] Sofia; [09:21] Sofia; [09:22] Diego |
| PRD-RISK-03 | docs/PRD.md | Risco | Latência depende de processamento e resposta externa | TRANSCRICAO | [09:07] Bruno; [09:09] Diego; [09:42] Diego |
| RFC-CTX-01 | docs/RFC.md | Contexto | Clientes usam polling e aceitam latência inferior a 10 segundos | TRANSCRICAO | [09:00] Marcos; [09:02] Marcos |
| RFC-CTX-02 | docs/RFC.md | Contexto | Transação de mudança de status atualiza pedido, histórico e estoque quando aplicável | CODIGO | src/modules/orders/order.service.ts |
| RFC-PROP-01 | docs/RFC.md | Decisão | Filtragem, outbox atômica e snapshot do evento | TRANSCRICAO | [09:34] Bruno; [09:40] Bruno; [09:52] Larissa |
| RFC-PROP-02 | docs/RFC.md | Decisão | Worker separado, polling a cada 2 segundos e conexão própria | TRANSCRICAO | [09:09] Diego; [09:11] Diego; [09:30] Bruno |
| RFC-PROP-03 | docs/RFC.md | Decisão | Entrega at-least-once e deduplicação pelo cliente | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Larissa |
| RFC-PROP-04 | docs/RFC.md | Requisito | API autenticada, URL e status configuráveis e customer explícito | TRANSCRICAO | [09:23] Sofia; [09:31] Marcos; [09:32] Larissa |
| RFC-ORDER-01 | docs/RFC.md | Restrição | Worker único e ordenação pretendida por pedido, não global | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| RFC-RES-01 | docs/RFC.md | Decisão | Retry limitado, DLQ separada, replay e histórico | TRANSCRICAO | [09:17] Larissa; [09:18] Diego; [09:34] Marcos; [09:36] Sofia |
| RFC-SEC-01 | docs/RFC.md | Segurança | HTTPS, HMAC, secrets por endpoint, rotação e replay por ADMIN | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia; [09:23] Sofia; [09:36] Sofia |
| RFC-INT-01 | docs/RFC.md | Decisão | Reutilizar padrões do projeto e processo separado | TRANSCRICAO | [09:27] Bruno; [09:29] Bruno; [09:30] Larissa |
| RFC-SCOPE-01 | docs/RFC.md | Exclusão | Sem webhook de entrada, email de falha ou dashboard | TRANSCRICAO | [09:02] Marcos; [09:37] Larissa; [09:40] Larissa |
| RFC-ALT-01 | docs/RFC.md | Alternativa | Envio síncrono rejeitado por impacto na transação | TRANSCRICAO | [09:04] Bruno; [09:06] Diego |
| RFC-ALT-02 | docs/RFC.md | Alternativa | Redis rejeitado por infraestrutura adicional | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Alternativa | Trigger MySQL não notifica diretamente o worker | TRANSCRICAO | [09:09] Bruno; [09:09] Diego |
| RFC-ALT-04 | docs/RFC.md | Alternativa | Exactly-once exigiria coordenação mais complexa | TRANSCRICAO | [09:25] Diego |
| RFC-OPEN-01 | docs/RFC.md | Questão em aberto | Observar e decidir rate limiting de saída | TRANSCRICAO | [09:39] Diego; [09:39] Larissa |
| RFC-OPEN-02 | docs/RFC.md | Questão em aberto | Contagem de cinco tentativas e cinco intervalos não esclarecida | TRANSCRICAO | [09:17] Larissa; [09:48] Larissa |
| RFC-OPEN-03 | docs/RFC.md | Questão em aberto | Condições de medição da meta de latência não especificadas | TRANSCRICAO | [09:02] Marcos; [09:09] Diego |
| RFC-OPEN-04 | docs/RFC.md | Questão em aberto | Autorização por cliente não decorre do JWT atual | CODIGO | src/middlewares/auth.middleware.ts; prisma/schema.prisma |
| RFC-OPEN-05 | docs/RFC.md | Questão em aberto | Ordenação por pedido em caso de retry ou replay não especificada | TRANSCRICAO | [09:12] Diego; [09:13] Larissa; [09:15] Diego; [09:18] Diego |
| RFC-RISK-01 | docs/RFC.md | Risco | Crescimento da outbox; arquivamento fora do escopo | TRANSCRICAO | [09:07] Bruno; [09:08] Diego |
| RFC-RISK-02 | docs/RFC.md | Risco | Proteção de dados e revisão de segurança antes do deploy | TRANSCRICAO | [09:19] Sofia; [09:22] Diego; [09:46] Sofia |
| FDD-CTX-01 | docs/FDD.md | Contexto | Transação de status e motivação da outbox | TRANSCRICAO | [09:04] Bruno; [09:06] Diego |
| FDD-ACT-01 | docs/FDD.md | Contexto | Usuário, integrador, administrador e worker | TRANSCRICAO | [09:11] Diego; [09:25] Diego; [09:31] Marcos; [09:36] Sofia |
| FDD-OPEN-01 | docs/FDD.md | Questão em aberto | Autorização, contagem de tentativas, latência e ordenação com retry | TRANSCRICAO | [09:02] Marcos; [09:12] Diego; [09:17] Larissa; [09:32] Larissa |
| FDD-TX-01 | docs/FDD.md | Decisão | Outbox filtrada e atômica com a mudança de status | TRANSCRICAO | [09:34] Bruno; [09:40] Bruno; [09:41] Diego |
| FDD-WORK-01 | docs/FDD.md | Decisão | Worker separado, polling e ordenação com um worker | TRANSCRICAO | [09:08] Diego; [09:09] Diego; [09:11] Diego; [09:12] Diego |
| FDD-SEC-01 | docs/FDD.md | Segurança | HTTPS, HMAC-SHA256 e secret por endpoint com rotação | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia; [09:23] Sofia |
| FDD-RETRY-01 | docs/FDD.md | Resiliência | Timeout, tentativas, backoff e DLQ | TRANSCRICAO | [09:17] Larissa; [09:18] Diego; [09:42] Diego |
| FDD-SCOPE-01 | docs/FDD.md | Requisito | CRUD, histórico, worker, retry e replay | TRANSCRICAO | [09:18] Diego; [09:31] Marcos; [09:33] Bruno; [09:34] Marcos; [09:36] Sofia |
| FDD-SCOPE-02 | docs/FDD.md | Exclusão | Entrada, email, dashboard, arquivamento, múltiplos workers e rate limiting | TRANSCRICAO | [09:02] Marcos; [09:08] Diego; [09:13] Diego; [09:37] Larissa; [09:39] Larissa; [09:40] Larissa |
| FDD-PAYLOAD-01 | docs/FDD.md | Contrato | Snapshot, dados e headers, limite de tamanho | TRANSCRICAO | [09:24] Larissa; [09:25] Diego; [09:43] Diego; [09:44] Diego; [09:44] Sofia; [09:52] Larissa |
| FDD-MODEL-01 | docs/FDD.md | Hipótese | Fan-out e identidade no replay propostos; fontes apenas fundamentam replay, UUID, filtros e snapshot | TRANSCRICAO | [09:18] Diego; [09:25] Diego; [09:33] Marcos; [09:52] Larissa |
| FDD-REPLAY-01 | docs/FDD.md | Requisito | Replay por ADMIN com auditoria | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| FDD-INTEG-01 | docs/FDD.md | Decisão | Padrões existentes e Prisma separado no worker | TRANSCRICAO | [09:28] Bruno; [09:29] Bruno; [09:30] Bruno; [09:30] Larissa |
| FDD-INTEG-02 | docs/FDD.md | Integração | Transação e validação de transição do pedido | CODIGO | src/modules/orders/order.service.ts; src/modules/orders/order.status.ts |
| FDD-INTEG-03 | docs/FDD.md | Integração | Composição da aplicação e prefixo das rotas | CODIGO | src/app.ts; src/routes/index.ts |
| FDD-AUTH-01 | docs/FDD.md | Questão em aberto | JWT e modelo atuais não estabelecem vínculo de usuário a cliente | CODIGO | src/middlewares/auth.middleware.ts; prisma/schema.prisma |
| FDD-ERR-01 | docs/FDD.md | Integração | Padrão de erros e tratamento atual de Zod | CODIGO | src/middlewares/error.middleware.ts; src/shared/errors/app-error.ts |
| FDD-API-01 | docs/FDD.md | Hipótese | Contratos propostos a partir de CRUD, histórico e replay discutidos; formatos não aprovados | TRANSCRICAO | [09:31] Marcos; [09:33] Bruno; [09:34] Marcos; [09:35] Diego |
| FDD-OBS-01 | docs/FDD.md | Hipótese | Métricas, tracing e alertas propostos; fontes fundamentam preocupações operacionais, Pino e auditoria | TRANSCRICAO | [09:07] Bruno; [09:18] Diego; [09:29] Bruno; [09:36] Sofia |
| FDD-DEPS-01 | docs/FDD.md | Dependência | Versões e ferramentas existentes | CODIGO | package.json; docker-compose.yml |
| FDD-RISK-01 | docs/FDD.md | Risco | Revisão de HMAC e geração de secret antes do deploy | TRANSCRICAO | [09:46] Sofia |
| ADR-001 | docs/adrs/ADR-001-outbox-transacional-mysql.md | Decisão | Outbox no MySQL, transação atômica e snapshot | TRANSCRICAO | [09:06] Diego; [09:08] Larissa; [09:34] Bruno; [09:40] Bruno; [09:52] Larissa |
| ADR-001-COD | docs/adrs/ADR-001-outbox-transacional-mysql.md | Contexto | Transação existente e datasource MySQL | CODIGO | src/modules/orders/order.service.ts; prisma/schema.prisma |
| ADR-002 | docs/adrs/ADR-002-worker-separado-polling.md | Decisão | Worker único separado e polling de 2 segundos | TRANSCRICAO | [09:09] Diego; [09:11] Diego; [09:12] Diego; [09:13] Larissa |
| ADR-003 | docs/adrs/ADR-003-retry-backoff-dlq.md | Decisão | Tentativas, backoff, DLQ separada e replay administrativo | TRANSCRICAO | [09:15] Diego; [09:17] Larissa; [09:18] Diego; [09:36] Sofia |
| ADR-004 | docs/adrs/ADR-004-hmac-e-rotacao-de-segredos.md | Decisão | HMAC-SHA256 e secret por endpoint com rotação de 24 horas | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia; [09:23] Sofia |
| ADR-005 | docs/adrs/ADR-005-at-least-once-e-deduplicacao.md | Decisão | At-least-once com UUID e deduplicação pelo cliente | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Larissa |
| ADR-006 | docs/adrs/ADR-006-reuso-de-padroes-existentes.md | Decisão | Reuso dos padrões de módulos, erros, validação e logging | TRANSCRICAO | [09:27] Bruno; [09:29] Bruno; [09:30] Larissa |
| ADR-006-COD | docs/adrs/ADR-006-reuso-de-padroes-existentes.md | Contexto | Organização modular e tratamento existente de erros e validação | CODIGO | src/modules/orders/order.routes.ts; src/middlewares/validate.middleware.ts; src/middlewares/error.middleware.ts |
| FDD-CONTRATO-01 | docs/FDD.md | Hipótese de contrato | Criação de configuração, retorno de secret e cliente informado; rota e resposta propostas | TRANSCRICAO | [09:31] Marcos; [09:32] Larissa |
| FDD-CONTRATO-02 | docs/FDD.md | Hipótese de contrato | Listagem por cliente; caminho, parâmetros e resposta propostos | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-03 | docs/FDD.md | Hipótese de contrato | Edição dos status e estado; caminho e resposta propostos | TRANSCRICAO | [09:21] Bruno; [09:33] Bruno; [09:33] Marcos |
| FDD-CONTRATO-04 | docs/FDD.md | Hipótese de contrato | Remoção de configuração; caminho e status HTTP propostos | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-05 | docs/FDD.md | Hipótese de contrato | Rotação da secret e 24 horas de sobreposição; caminho e resposta propostos | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia |
| FDD-CONTRATO-06 | docs/FDD.md | Contrato | Histórico dos últimos 100 envios e dados solicitados; resposta detalhada proposta | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-07 | docs/FDD.md | Contrato | Caminho de replay, role ADMIN e auditoria; corpo e status HTTP propostos | TRANSCRICAO | [09:35] Diego; [09:36] Sofia |
| FDD-CONTRATO-08 | docs/FDD.md | Contrato | JSON, cabeçalhos e assinatura do webhook; codificação exata e resposta do receptor pendentes | TRANSCRICAO | [09:20] Sofia; [09:25] Diego; [09:43] Diego; [09:44] Diego; [09:44] Sofia |
| FDD-CONTRATO-09 | docs/FDD.md | Compatibilidade | Prefixo da API existente; caminhos finais de webhooks permanecem propostos | CODIGO | src/app.ts; src/routes/index.ts |
| FDD-FLUXO-01 | docs/FDD.md | Hipótese | Recuperação após interrupção e reivindicação de trabalho; fontes descrevem worker e polling, não o mecanismo | TRANSCRICAO | [09:09] Diego; [09:11] Diego |
| FDD-FLUXO-02 | docs/FDD.md | Hipótese | Atomicidade da transição para DLQ proposta; fonte descreve tabela separada | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-03 | docs/FDD.md | Questão em aberto | Classificação de respostas e tratamento de payload acima de 64 KB não definidos | TRANSCRICAO | [09:23] Sofia; [09:24] Larissa; [09:42] Diego |
| FDD-ERROS-02 | docs/FDD.md | Hipótese | Matriz WEBHOOK_* e códigos HTTP propostos, apoiados no padrão de erros | TRANSCRICAO | [09:28] Bruno; [09:29] Larissa |
| FDD-ERROS-03 | docs/FDD.md | Integração | Envelope de erros AppError e códigos genéricos existentes | CODIGO | src/shared/errors/app-error.ts; src/middlewares/error.middleware.ts |
| FDD-TESTE-01 | docs/FDD.md | Critério de aceite | Commit e rollback de status, histórico, estoque e outbox elegível | TRANSCRICAO | [09:34] Bruno; [09:40] Bruno; [09:41] Diego |
| FDD-TESTE-02 | docs/FDD.md | Critério de aceite | Validar autenticação, assinatura, limite de payload e rotação | TRANSCRICAO | [09:22] Sofia; [09:23] Sofia; [09:24] Larissa; [09:36] Sofia |
| FDD-TESTE-03 | docs/FDD.md | Critério de aceite | Testar retry, DLQ e replay ADMIN auditado; detalhes dependem de definição | TRANSCRICAO | [09:17] Larissa; [09:18] Diego; [09:36] Sofia; [09:42] Diego |
| FDD-RISCO-02 | docs/FDD.md | Risco | Isolamento entre clientes não está estabelecido pelo JWT e modelo atuais | CODIGO | src/middlewares/auth.middleware.ts; prisma/schema.prisma |
| FDD-RISCO-03 | docs/FDD.md | Risco | Backlog e reordenação em falhas; interação com retry requer definição | TRANSCRICAO | [09:07] Bruno; [09:12] Diego; [09:13] Larissa; [09:15] Diego |
| FDD-RISCO-04 | docs/FDD.md | Risco | Vazamento de secret e verificação de origem e integridade | TRANSCRICAO | [09:19] Sofia; [09:21] Sofia; [09:22] Diego |
| ADR-001-ALT-01 | docs/adrs/ADR-001-outbox-transacional-mysql.md | Alternativa | Envio síncrono descartado por acoplamento à disponibilidade do cliente | TRANSCRICAO | [09:04] Bruno; [09:06] Diego |
| ADR-001-ALT-02 | docs/adrs/ADR-001-outbox-transacional-mysql.md | Alternativa | Redis descartado por infraestrutura adicional | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| ADR-001-ALT-03 | docs/adrs/ADR-001-outbox-transacional-mysql.md | Alternativa | Snapshot escolhido em vez de renderização tardia | TRANSCRICAO | [09:51] Bruno; [09:52] Larissa |
| ADR-001-RISCO-01 | docs/adrs/ADR-001-outbox-transacional-mysql.md | Risco | Crescimento da tabela e arquivamento fora de escopo | TRANSCRICAO | [09:07] Bruno; [09:08] Diego |
| ADR-002-ALT-01 | docs/adrs/ADR-002-worker-separado-polling.md | Alternativa | Trigger não notifica diretamente processo externo | TRANSCRICAO | [09:09] Bruno; [09:09] Diego |
| ADR-002-ALT-02 | docs/adrs/ADR-002-worker-separado-polling.md | Alternativa | Worker no mesmo processo afetado por reinício da API | TRANSCRICAO | [09:11] Diego |
| ADR-002-OPEN-01 | docs/adrs/ADR-002-worker-separado-polling.md | Questão em aberto | Ordenação em retries e condição de latência não especificadas | TRANSCRICAO | [09:02] Marcos; [09:12] Diego; [09:13] Larissa; [09:15] Diego |
| ADR-002-COD-01 | docs/adrs/ADR-002-worker-separado-polling.md | Contexto | Entradas e composição atuais da API, sem worker | CODIGO | src/server.ts; src/app.ts |
| ADR-003-ALT-01 | docs/adrs/ADR-003-retry-backoff-dlq.md | Alternativa | Três tentativas e retry indefinido não foram escolhidos | TRANSCRICAO | [09:15] Diego; [09:16] Bruno; [09:16] Diego |
| ADR-003-ALT-02 | docs/adrs/ADR-003-retry-backoff-dlq.md | Alternativa | Marcar falha na outbox em vez de DLQ separada | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| ADR-003-OPEN-01 | docs/adrs/ADR-003-retry-backoff-dlq.md | Questão em aberto | Número de tentativas e sequência dos cinco intervalos | TRANSCRICAO | [09:17] Larissa; [09:48] Larissa |
| ADR-004-ALT-01 | docs/adrs/ADR-004-hmac-e-rotacao-de-segredos.md | Alternativa | Secret global descartada por ampliar impacto de vazamento | TRANSCRICAO | [09:21] Sofia |
| ADR-004-OPEN-01 | docs/adrs/ADR-004-hmac-e-rotacao-de-segredos.md | Questão em aberto | Formato da assinatura e seleção de secrets na sobreposição não especificados | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia |
| ADR-004-OPEN-02 | docs/adrs/ADR-004-hmac-e-rotacao-de-segredos.md | Questão em aberto | Timestamp citado para replay; autenticação do timestamp não especificada | TRANSCRICAO | [09:22] Sofia; [09:44] Diego |
| ADR-005-ALT-01 | docs/adrs/ADR-005-at-least-once-e-deduplicacao.md | Alternativa | Exactly-once descartado por complexidade de coordenação | TRANSCRICAO | [09:25] Diego |
| ADR-005-OPEN-01 | docs/adrs/ADR-005-at-least-once-e-deduplicacao.md | Questão em aberto | Identidade no replay e em múltiplos endpoints não especificada | TRANSCRICAO | [09:18] Diego; [09:25] Diego; [09:33] Marcos |
| ADR-006-VALID-01 | docs/adrs/ADR-006-reuso-de-padroes-existentes.md | Compatibilidade | Código genérico de validação existente é VALIDATION_ERROR | CODIGO | src/middlewares/validate.middleware.ts; src/middlewares/error.middleware.ts; src/shared/errors/http-errors.ts |
| ADR-006-WORK-01 | docs/adrs/ADR-006-reuso-de-padroes-existentes.md | Decisão | Worker separado usa cliente Prisma próprio e mesmo banco | TRANSCRICAO | [09:28] Bruno; [09:30] Bruno |
