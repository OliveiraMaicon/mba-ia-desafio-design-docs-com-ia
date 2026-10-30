# PRD — Sistema de Webhooks de Notificação de Pedidos

**Status:** proposta para revisão · **Fonte:** [transcrição da reunião](../TRANSCRICAO.md) e código existente · **Rastreabilidade:** [tracker](TRACKER.md)

## 1. Resumo executivo

### Resumo e contexto da feature

O OMS não dispõe de notificações externas. Atlas Comercial, MaxDistribuição e Nova Cargo solicitaram avisos quando o status de seus pedidos mudar. Hoje consultam repetidamente a API de pedidos, o que torna a integração lenta e custosa; a Atlas indicou risco de migração para um concorrente se a necessidade não for atendida [PRD-CTX-01].

A proposta é disponibilizar webhooks de saída configuráveis por cliente, com registro transacional dos eventos, envio assíncrono, assinatura e recuperação de falhas. O escopo é documental: a feature descrita aqui ainda não existe no código [PRD-CTX-02].

### Problema e motivação

O polling transfere aos clientes o custo de detectar mudanças e não garante que eles as percebam rapidamente. A solução deve reduzir essa dependência sem fazer a mudança de status aguardar uma chamada HTTP externa [PRD-CTX-01, PRD-DEC-01].

### Objetivos e métricas de sucesso

| Objetivo | Indicador e meta | Condições e origem |
| --- | --- | --- |
| Notificação próxima do tempo real | Abaixo de **10 segundos** | Expectativa dos clientes; início e fim da medição, percentil, volume e condições de operação ainda precisam ser definidos. O polling de 2 s, isoladamente, não garante a meta [PRD-MET-01]. |
| Recuperar indisponibilidades temporárias | Política acordada de **5 tentativas** e intervalos de 1 min, 5 min, 30 min, 2 h e 12 h; depois, DLQ | A reunião não esclarece se a tentativa inicial integra as cinco; alinhar antes da implementação [PRD-RF-06]. |
| Facilitar diagnóstico | Histórico dos **últimos 100** envios por webhook, com sucesso/falha, payload, resposta e duração | Funcionalidade solicitada para consulta do cliente [PRD-RF-08]. |
| Viabilizar prazo do cliente | Estimativa de **3 sprints**, com revisão de segurança de pelo menos **2 dias úteis** antes do deploy | A Atlas pediu fim de novembro; Marcos ficou de confirmar o prazo. Não há ano ou data de compromisso definidos na transcrição [PRD-PLAN-01]. |

## 2. Experiência e funcionalidades

### Público-alvo e cenários de uso

- **Integrador B2B:** configura uma URL HTTPS e os status de interesse; recebe e valida notificações e elimina duplicatas por identificador do evento [PRD-RF-01, PRD-RF-02, PRD-RF-05].
- **Cliente que acompanha integrações:** consulta o histórico de entregas para identificar falhas e respostas do seu endpoint [PRD-RF-08].
- **Administrador do OMS:** investiga falhas permanentes e solicita o replay de um evento da DLQ, com registro de autoria [PRD-RF-09].

### Escopo incluso e requisitos funcionais

| ID | Requisito funcional |
| --- | --- |
| PRD-RF-01 | Permitir que usuário autenticado cadastre webhook para um `customer_id` informado no corpo ou caminho, com URL e lista de status; gerar e devolver uma secret na criação. O `customer_id` não vem do JWT atual. |
| PRD-RF-02 | Permitir editar, remover e listar configurações de webhooks de um cliente; cada endpoint tem sua própria seleção de status. |
| PRD-RF-03 | Ao mudar o status de um pedido, filtrar as configurações interessadas antes de inserir na outbox; se nenhuma estiver interessada, não criar evento de webhook. |
| PRD-RF-04 | Registrar o evento elegível na mesma transação da mudança de status e do histórico, com payload contendo um snapshot do momento da mudança; falha na inserção deve reverter a transação. |
| PRD-RF-05 | Enviar notificações de saída com identificador único do evento para deduplicação pelo cliente; a entrega pode ocorrer mais de uma vez. |
| PRD-RF-06 | Em caso de falha, aplicar a política de 5 tentativas e os intervalos acordados de 1 min, 5 min, 30 min, 2 h e 12 h; ao esgotar a política, registrar em DLQ separada, com payload, motivo e timestamp. A contagem exata permanece em aberto. |
| PRD-RF-07 | Permitir rotação da secret por endpoint, mantendo a anterior válida em paralelo por 24 horas. |
| PRD-RF-08 | Exibir os últimos 100 envios de um webhook, com resultado, payload, resposta e tempo de resposta. |
| PRD-RF-09 | Permitir replay manual de evento da DLQ por endpoint restrito à role `ADMIN`, registrando quem solicitou; o replay recoloca o evento como pendente na outbox. |
| PRD-RF-10 | Disponibilizar no portal de desenvolvedor orientação para integrar via API e deduplicar usando o identificador do evento. |

### Fora de escopo

- Receber webhooks de clientes: a integração desta fase é somente de saída [PRD-SCOPE-01].
- Email ou outra notificação por email em caso de falhas: adiado para possível fase posterior [PRD-SCOPE-02].
- Dashboard visual: projeto separado do time de frontend; nesta fase há apenas endpoints [PRD-SCOPE-03].
- Rate limiting de envio: observar o comportamento e decidir posteriormente; não é compromisso da fase [PRD-SCOPE-04].
- Arquivamento de eventos entregues e escalabilidade com múltiplos workers: fora desta fase; não há garantia de ordenação global [PRD-SCOPE-05].

### Critérios de aceitação

- **Configuração:** usuário autenticado cadastra, edita, lista e remove webhooks; a criação retorna secret e rejeita URL HTTP. O cliente é informado explicitamente, não extraído do JWT [PRD-RF-01, PRD-RF-02, PRD-NFR-01].
- **Evento:** para status elegível, a mudança e o registro na outbox são confirmados ou revertidos juntos; sem webhook interessado, não há registro. O payload preserva o snapshot mesmo após alterações posteriores no pedido [PRD-RF-03, PRD-RF-04].
- **Entrega e segurança:** o cliente recebe JSON assinado, consegue identificar o evento e o webhook e deduplicar reentregas. A rotação mantém a secret anterior válida por 24 horas [PRD-RF-05, PRD-RF-07, PRD-NFR-01, PRD-NFR-02].
- **Falha e operação:** timeout é tratado como falha; após a política de tentativas acordada, o evento vai à DLQ. O histórico apresenta os dados solicitados e apenas `ADMIN` pode fazer replay, com auditoria [PRD-RF-06, PRD-RF-08, PRD-RF-09, PRD-NFR-03].
- **Latência:** validar a meta abaixo de 10 segundos somente após definir janela, percentil e condições de medição; não confundir o intervalo de polling com garantia de entrega [PRD-MET-01].

## 3. Especificações técnicas

### Decisões e trade-offs principais

| Decisão | Trade-off |
| --- | --- |
| Outbox no MySQL na transação de status; worker separado consulta pendências a cada 2 s [PRD-DEC-01]. | Evita chamada síncrona e infraestrutura Redis adicional, mas introduz polling e operação de outro processo. |
| Processamento por um único worker, com ordenação pretendida por pedido; sem ordenação global [PRD-DEC-02]. | Simplicidade agora, mas múltiplos workers exigiriam outra estratégia. A reunião não resolveu como preservar ordem diante de falhas e retries. |
| Entrega *at-least-once* com `X-Event-Id` [PRD-RF-05]. | Evita a complexidade de *exactly-once*, mas exige deduplicação no cliente. O limite de tentativas e a DLQ significam que entrega bem-sucedida não é garantida indefinidamente. |
| HMAC-SHA256 e secret única por endpoint, com rotação [PRD-NFR-01]. | Permite verificar autenticidade e integridade e limita o impacto de vazamento, mas exige gestão de secrets dos dois lados. |

### Requisitos não funcionais

- **PRD-NFR-01 — Segurança:** apenas HTTPS; assinatura HMAC-SHA256 sobre o corpo, secret única por endpoint e rotação com 24 h de sobreposição. CRUD exige autenticação; replay exige `ADMIN`.
- **PRD-NFR-02 — Contrato:** JSON com `event_id`, `event_type`, timestamp ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e dados básicos como `total_cents`, sem itens. Headers: `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type`.
- **PRD-NFR-03 — Limites:** payload acima de 64 KB gera erro e não é enviado; timeout HTTP de 10 s é tratado como falha para retry.
- **PRD-NFR-04 — Compatibilidade:** reutilizar os padrões de módulos, schemas Zod, erros `WEBHOOK_*`, `AppError`, middleware centralizado e logger Pino. O worker é outro processo Node, com instância própria de PrismaClient e o mesmo banco.

### Dependências e integração

- O MySQL e o Prisma existentes são a base para a outbox e DLQ; `src/modules/orders/order.service.ts` concentra a transação de `changeStatus`, incluindo atualização de estoque e histórico [PRD-DEP-01].
- `src/middlewares/auth.middleware.ts` implementa autenticação JWT e `requireRole`; o usuário autenticado tem `id`, `email` e `role`, não `customer_id`. O vínculo e a autorização por cliente não estão definidos na reunião nem no modelo atual; devem ser esclarecidos antes de expor dados de clientes [PRD-DEP-02].
- O integrador precisa disponibilizar endpoint HTTPS, validar a assinatura e tratar duplicatas. A revisão de Sofia e a documentação para clientes precedem a disponibilização [PRD-PLAN-01, PRD-RF-10].

## 4. Riscos e planejamento

### Riscos e mitigação

Probabilidade e impacto abaixo são avaliações qualitativas para revisão, não estimativas apresentadas na reunião.

| Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- |
| Endpoint externo indisponível ou lento [PRD-RISK-01] | A avaliar | Alto | Timeout, retries e DLQ com replay administrativo; consultar histórico. |
| Secret vazada ou payload adulterado [PRD-RISK-02] | A avaliar | Alto | HMAC por endpoint, rotação com sobreposição e revisão de segurança antes do deploy. |
| Latência acima de 10 s com backlog, retries ou cliente lento [PRD-RISK-03] | A avaliar | Alto | Medir latência e carga; validar capacidade do worker e explicitar condições da meta. O polling de 2 s não limita o tempo total. |
| Acesso indevido entre clientes [PRD-DEP-02] | A avaliar | Alto | Definir e revisar a autorização por cliente antes da disponibilização; JWT atual não informa vínculo com customer. |
| Eventos do mesmo pedido recebidos fora de ordem após falha e retry [PRD-DEC-02] | A avaliar | A avaliar | Validar o comportamento e decidir a política de ordenação antes de prometer garantia ao cliente. |

### Planejamento e questões em aberto

- Estimativa de três sprints: modelagem, worker/retry, configuração/histórico, integração e testes; incluir pelo menos dois dias úteis de revisão de segurança antes do deploy. A data solicitada pela Atlas ainda precisa ser confirmada [PRD-PLAN-01].
- Email e dashboard são possíveis trabalhos posteriores, sem cronograma aprovado; rate limiting depende de observação [PRD-SCOPE-02, PRD-SCOPE-03, PRD-SCOPE-04].
- Esclarecer se as cinco tentativas incluem a primeira chamada e como os cinco intervalos se aplicam [PRD-RF-06].
- Definir medição e condições da meta de latência, autorização por cliente e ordenação com retries; não presumir essas respostas como decisões tomadas [PRD-MET-01, PRD-DEP-02, PRD-DEC-02].

### Estratégia de testes e validação

- Exercitar CRUD autenticado, filtro por cliente/status, rejeição de HTTP e restrição de replay a `ADMIN`; verificar também o registro de autoria [PRD-RF-01, PRD-RF-02, PRD-RF-03, PRD-RF-09, PRD-NFR-01].
- Testar commit e rollback da transação de mudança de status/outbox e preservação do snapshot; conferir o contrato JSON e a deduplicação por `X-Event-Id` [PRD-RF-04, PRD-RF-05, PRD-NFR-02].
- Simular indisponibilidade, timeout, retries, DLQ e replay; validar histórico e limite de payload. Confirmar a semântica das tentativas antes de fixar a contagem esperada [PRD-RF-06, PRD-RF-08, PRD-RF-09, PRD-NFR-03].
- Verificar assinatura, adulteração, secrets distintas e janela de rotação; submeter a implementação à revisão de Sofia. Medir latência sob condições a definir e testar o comportamento de ordenação em falhas [PRD-NFR-01, PRD-PLAN-01, PRD-MET-01, PRD-DEC-02].
