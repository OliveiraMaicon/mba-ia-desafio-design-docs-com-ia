# RFC — Webhooks de notificação de pedidos

| Metadado | Valor |
| --- | --- |
| Autor | OpenCode, com base na reunião técnica |
| Status | Em revisão — decisões da reunião documentadas, questões pendentes de validação |
| Data | 2026-10-30 |
| Revisores propostos | Larissa (Tech Lead), Marcos (Produto), Bruno (Pedidos), Diego (Plataforma) e Sofia (Segurança) |
| Documentos relacionados | [PRD](PRD.md) · [FDD em elaboração](FDD.md) · [Tracker](TRACKER.md) · [Transcrição](../TRANSCRICAO.md) |

## Resumo executivo — TL;DR

Propõe-se notificar clientes B2B sobre mudanças de status por webhooks de saída. Eventos elegíveis serão persistidos em uma **outbox no MySQL**, na mesma transação da mudança de status; um **worker separado** fará polling e tentará a entrega via HTTPS. Falhas seguem uma política limitada de retry e, ao esgotá-la, vão para uma DLQ com replay administrativo [RFC-PROP-01, RFC-PROP-02].

As mensagens serão assinadas por endpoint e poderão ser entregues mais de uma vez: o integrador deverá deduplicá-las usando o identificador do evento. A solução evita bloquear a transação de pedidos por uma chamada externa e dispensa infraestrutura de mensageria adicional, mas exige operação do worker e tratamento de duplicatas pelo cliente [RFC-PROP-03, RFC-ALT-01, RFC-ALT-02].

## Contexto e problema

Atlas Comercial, MaxDistribuição e Nova Cargo consultam periodicamente os pedidos para identificar alterações, tornando suas integrações lentas e custosas. Os clientes consideram menos de 10 segundos aceitável como tempo real; a forma de medir essa expectativa ainda precisa ser definida [RFC-CTX-01].

No OMS atual, `OrderService.changeStatus` já concentra em uma transação a alteração do pedido, o histórico e, quando aplicável, o estoque. Acrescentar uma chamada HTTP síncrona vincularia essa operação à disponibilidade e à latência do cliente. O sistema atual não implementa webhooks [RFC-CTX-02, RFC-ALT-01].

## Proposta técnica

### Fluxo e limites

1. O cliente configura, via API autenticada, um endpoint HTTPS e os status de interesse. A identificação do cliente é informada na requisição, não derivada do JWT atual [RFC-PROP-04].
2. Em uma mudança de status, o sistema verifica os webhooks interessados; se houver, registra um **snapshot do evento** na outbox dentro da mesma transação do pedido. Uma falha nessa gravação reverte a mudança; sem interessados, não se cria evento [RFC-PROP-01].
3. Um único worker em processo separado consulta a outbox a cada 2 segundos e envia os eventos elegíveis aos endpoints. Ele usa o mesmo banco, mas uma instância própria do PrismaClient. A intenção é preservar a ordem por pedido com um único worker, sem prometer ordem global [RFC-PROP-02, RFC-ORDER-01].
4. As falhas são retentadas com os intervalos acordados; após esgotar a política, o evento é persistido em uma DLQ separada. Um administrador pode solicitar replay, com auditoria. O cliente também poderá consultar o histórico de entregas [RFC-RES-01].

O sistema fornece entrega **at-least-once** no sentido de permitir reentregas, não uma promessa de sucesso indefinido: após o limite de tentativas o evento vai à DLQ. A resposta a falhas, os contratos e as decisões de implementação devem ser detalhados no FDD [RFC-PROP-03, RFC-RES-01].

### Segurança e integração

O payload será assinado com **HMAC-SHA256**, com secret específica por endpoint e rotação com sobreposição de 24 horas. O identificador do evento permite deduplicação no cliente; HTTPS é obrigatório. O replay exige role `ADMIN` e registro de autoria [RFC-SEC-01].

O módulo seguirá os padrões existentes de controllers, services, repositories, schemas Zod, `AppError`, middleware de erro e Pino; os novos códigos de erro usarão `WEBHOOK_`. O worker será independente do processo da API [RFC-INT-01].

Nesta fase não são propostos webhooks de entrada, emails de falha ou dashboard visual. Rate limiting de saída permanece para observação e decisão posterior [RFC-SCOPE-01, RFC-OPEN-01].

## Alternativas consideradas

| Alternativa discutida | Motivo do descarte e trade-off |
| --- | --- |
| Envio HTTP síncrono durante a mudança de status | Um endpoint lento ou indisponível afetaria a transação de pedidos; a outbox desacopla o envio, ao custo de processamento e operação assíncronos [RFC-ALT-01]. |
| Redis Streams / infraestrutura Redis para entrega | Introduziria infraestrutura adicional para um time pequeno; a outbox reutiliza o MySQL, ao custo de polling e gestão da tabela [RFC-ALT-02]. |
| Trigger do MySQL para acordar o worker | A trigger executa SQL, mas não notifica diretamente um processo externo; o polling é mais simples, com latência de consulta e carga periódica [RFC-ALT-03]. |
| Garantia *exactly-once* | Exigiria coordenação mais complexa entre emissor e receptor; *at-least-once* simplifica o envio, transferindo a deduplicação ao integrador [RFC-ALT-04]. |

## Questões em aberto para revisão

| ID | Questão | Consequência para a revisão |
| --- | --- | --- |
| RFC-OPEN-01 | Qual necessidade real de rate limiting de saída por cliente? | Observar o tráfego e decidir depois; não incluir como capacidade já definida. |
| RFC-OPEN-02 | As cinco tentativas incluem a primeira entrega? Como aplicar os cinco intervalos acordados? | A reunião enumera cinco tentativas e cinco intervalos, mas não resolve essa contagem; definir antes da implementação. |
| RFC-OPEN-03 | Como definir e medir a meta de menos de 10 segundos? | O polling de 2 s não garante latência ponta a ponta; faltam janela, percentil e condições de carga e disponibilidade. |
| RFC-OPEN-04 | Como autorizar cada usuário a operar dados de determinado cliente? | O JWT existente não contém `customer_id`; aceitar um ID fornecido na requisição, por si só, não comprova permissão de acesso. A reunião permite CRUD a roles autenticadas, mas não define o vínculo por cliente. |
| RFC-OPEN-05 | Como tratar ordenação por pedido quando um envio falha e outro está pronto, inclusive após replay? | A reunião pretende ordenação por pedido com um worker, mas não especifica a interação com retries; validar antes de prometer essa garantia. |

## Impacto e riscos

- **Pedidos e banco:** a outbox torna o evento elegível atômico com a transação, mas indisponibilidade de sua gravação também impede a mudança de status. O crescimento da tabela e seu arquivamento posterior precisam ser considerados [RFC-PROP-01, RFC-RISK-01].
- **Operação:** um processo adicional precisa ser executado e acompanhado; backlog, cliente lento e retries podem elevar a latência acima da expectativa. A política limitada e a DLQ requerem acompanhamento operacional [RFC-PROP-02, RFC-RES-01, RFC-OPEN-03].
- **Clientes:** integrar exige endpoint HTTPS, verificação da assinatura e deduplicação. Falhas persistentes não são resolvidas automaticamente após a DLQ [RFC-PROP-03, RFC-SEC-01, RFC-RES-01].
- **Segurança:** a autorização por cliente ainda está aberta; exposição de secrets e dados de pedidos exige revisão. Sofia solicitou pelo menos dois dias úteis de revisão de segurança antes do deploy [RFC-OPEN-04, RFC-RISK-02].

## Decisões relacionadas

Os ADRs registram decisões acordadas na reunião; a feature ainda não está implementada:

- [ADR-001: Outbox transacional no MySQL](adrs/ADR-001-outbox-transacional-mysql.md) [RFC-PROP-01].
- [ADR-002: Worker separado com polling](adrs/ADR-002-worker-separado-polling.md) [RFC-PROP-02].
- [ADR-003: Retry com backoff e DLQ](adrs/ADR-003-retry-backoff-dlq.md) [RFC-RES-01].
- [ADR-004: HMAC e rotação de segredos](adrs/ADR-004-hmac-e-rotacao-de-segredos.md) [RFC-SEC-01].
- [ADR-005: At-least-once e deduplicação](adrs/ADR-005-at-least-once-e-deduplicacao.md) [RFC-PROP-03].
- [ADR-006: Reuso dos padrões existentes](adrs/ADR-006-reuso-de-padroes-existentes.md) [RFC-INT-01].
