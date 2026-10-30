### FDD: Webhooks de notificação de pedidos

Versão: 0.1, proposta para revisão

Data: 2026-10-30

Responsável: a definir; revisão técnica por Larissa, Bruno, Diego e Sofia

As decisões da reunião e os fatos do código são rastreados no [tracker](TRACKER.md). **Hipótese** identifica proposta de implementação não decidida na reunião, sujeita a revisão; não é requisito aprovado. Este documento não autoriza implementar a feature como parte do desafio.

---

### 1. Contexto e motivação técnica

O OMS ainda não envia notificações externas. A mudança de status já atualiza pedido, histórico e estoque em uma transação; uma chamada HTTP externa dentro dela acoplaria a operação à disponibilidade do cliente. A [RFC](RFC.md) propõe outbox MySQL transacional e worker independente para notificar endpoints de clientes B2B [FDD-CTX-01].

**Atores:** usuário autenticado configura webhooks; sistema receptor valida e deduplica eventos; administrador reprocessa falhas; worker envia eventos [FDD-ACT-01].

**Bloqueios para implementação pública:** a autorização de usuário por cliente não está definida; a reunião não resolve a contagem das cinco tentativas, a medição da meta de latência nem a ordenação em retries. Não apresentar hipóteses deste FDD como decisões fechadas [FDD-OPEN-01].

---

### 2. Objetivos técnicos

- Persistir o evento elegível e o status na mesma transação; se a inserção falhar, reverter também histórico e estoque [FDD-TX-01].
- Consultar a outbox a cada 2 segundos em processo separado; expectativa do cliente inferior a 10 segundos, ainda sem percentil ou condições definidos. O polling não garante latência ponta a ponta [FDD-WORK-01, FDD-OPEN-01].
- Assinar o corpo com HMAC-SHA256, usar HTTPS e permitir deduplicação via `X-Event-Id`; entregas podem ser repetidas e falhas permanentes seguem para DLQ [FDD-SEC-01, FDD-RETRY-01].

---

### 3. Escopo e exclusões

**Incluído**
- CRUD de configuração, filtro por status, rotação de secret, histórico de entregas, outbox transacional, worker, retry, DLQ e replay auditado por `ADMIN` [FDD-SCOPE-01].
- Payload enxuto com snapshot, autenticação, reutilização dos padrões do OMS e documentação de integração para clientes [FDD-PAYLOAD-01, FDD-INTEG-01].

**Excluído**
- Webhooks de entrada, notificação por email, dashboard visual, arquivamento e múltiplos workers nesta fase. Rate limiting de saída fica para observação e decisão posterior [FDD-SCOPE-02].

---

### 4. Fluxos detalhados e diagramas

**Modelo lógico, hipótese para revisão:** `webhook_config` associa cliente, URL, status, estado e secrets; `webhook_outbox` guarda UUID do evento, pedido, snapshot, timestamp e estado; uma entrega por endpoint guarda estado, tentativas e próximo horário; `webhook_dead_letter` guarda payload, motivo e timestamp. Um mesmo evento pode ter várias entregas, cada uma com falha independente. Preservar o `event_id` em retry e replay; um receptor que já processou o evento pode ignorar o replay por deduplicação. Identidade por evento, fan-out e replay são propostas, não decisões da reunião [FDD-MODEL-01].

**Fluxo principal**
1. Na chamada autenticada de mudança de status, validar transição existente; dentro de `prisma.$transaction`, atualizar estoque quando necessário, pedido e histórico [FDD-CTX-01].
2. No mesmo `tx`, selecionar configurações ativas do cliente interessadas no novo status. Sem correspondência, não inserir na outbox. Havendo correspondência, gerar UUID e snapshot JSON do evento, persistindo-o e associando as entregas elegíveis. Erro de persistência propaga e reverte a transação [FDD-TX-01, FDD-MODEL-01].
3. Um único worker em processo próprio consulta pendências mais antigas em pequenos lotes, a cada 2 segundos. **Hipótese:** persistir reivindicação e recuperar trabalho interrompido após reinício; tempo de recuperação e mecanismo de exclusão mútua dependem de revisão. Não usar dois workers sem nova decisão [FDD-WORK-01].
4. Preparar corpo e assinatura para o endpoint, verificar limite de 64 KB e enviar via HTTPS. O timeout é 10 segundos; persistir resultado e duração no histórico. **Hipótese:** tratar respostas 2xx como sucesso; classificação das demais respostas deve ser aprovada [FDD-PAYLOAD-01, FDD-RETRY-01].

**Fluxos alternativos e exceções**
- Em timeout ou falha de envio, programar retry sem reexecutar a transação do pedido. A reunião registra cinco tentativas e intervalos 1 min, 5 min, 30 min, 2 h, 12 h, mas não esclarece se a chamada inicial está incluída. Definir essa contagem antes de implementar o escalonamento; não inventar uma sexta chamada [FDD-RETRY-01].
- Esgotada a política, mover a entrega para DLQ persistente, com payload, motivo e timestamp. **Hipótese:** a mudança de estado e a criação da DLQ devem ser atômicas para evitar perda em falhas de processo [FDD-RETRY-01].
- Replay exige `ADMIN` e auditoria do usuário; recoloca a entrega como pendente. A decisão sobre contadores, concorrência entre replays e transição da DLQ permanece pendente [FDD-REPLAY-01].
- Rotacionar secret com validade paralela de 24 h para a anterior. **Pendente:** definir formato de assinatura durante a sobreposição e procedimento exato de validação para o receptor, pois a reunião não os especifica [FDD-SEC-01].
- Payload maior que 64 KB resulta em erro e não é enviado; seu tratamento operacional e relação com DLQ ainda precisam de decisão [FDD-PAYLOAD-01].

**Diagrama**
```text
PATCH status -> transação: pedido + histórico + estoque + outbox elegível
                                               |
                                               v
                                  worker independente (polling 2 s)
                                               |
                              HTTPS assinado -> sucesso / retry -> DLQ
                                                                  |
                                                         replay por ADMIN
```

**Integração com o sistema existente**
- `src/modules/orders/order.service.ts`: em `changeStatus`, chamar função proposta `publishWebhookEvent(tx, order, fromStatus, toStatus)` dentro do callback transacional, após validar a transição. Nenhum HTTP deve ocorrer na transação [FDD-INTEG-02].
- `src/modules/orders/order.status.ts`: reutilizar `canTransition` e regras de estoque; eventos seguem apenas mudanças de status aceitas [FDD-INTEG-02].
- `src/app.ts` e `src/routes/index.ts`: conectar controller, service e repository do novo módulo e registrar router sob `/api/v1`; os arquivos atuais são pontos reais de composição [FDD-INTEG-03].
- `src/middlewares/auth.middleware.ts`: reutilizar `authenticate` e `requireRole('ADMIN')` para replay. O JWT não contém `customer_id`; **hipótese**: validar vínculo usuário-cliente em camada dedicada. Modelo e política do vínculo precisam de aprovação antes da disponibilização [FDD-AUTH-01].
- `src/middlewares/validate.middleware.ts`, `src/shared/errors/app-error.ts` e `src/middlewares/error.middleware.ts`: integrar schemas Zod e erros específicos `WEBHOOK_*`. Atualmente Zod gera `VALIDATION_ERROR`; para o prefixo específico, mapear erros do módulo explicitamente para `AppError`, sem alterar o tratamento global de outras rotas [FDD-ERR-01].
- `src/shared/logger/index.ts` e `src/config/database.ts`: reutilizar Pino e configuração do Prisma; worker abre seu próprio client por ser outro processo, apontando ao mesmo banco [FDD-INTEG-01].
- `prisma/schema.prisma`: propor modelos de configuração, outbox, entregas e DLQ; campos, índices e migração devem ser revisados antes de implementação [FDD-MODEL-01].

---

### 5. Contratos públicos (assinaturas, endpoints, headers, exemplos)

Os caminhos e formatos abaixo são **hipóteses de contrato**, exceto os caminhos de histórico e replay citados na reunião. Prefixo `/api/v1` segue o roteador atual. Todos os exemplos de secret e UUID são fictícios; contratos dependem de revisão, inclusive autorização por cliente [FDD-API-01].

**Criar configuração**

Tipo: endpoint; método: `POST /api/v1/webhooks`. Header: `Authorization: Bearer <JWT>`; `Content-Type: application/json`. `customerId` não é inferido do token; `statuses` contém status de interesse.

Requisição:
```json
{"customerId":"11111111-1111-4111-8111-111111111111","url":"https://cliente.example/webhooks","statuses":["SHIPPED","DELIVERED"]}
```
Resposta proposta `201 Created`:
```json
{"id":"22222222-2222-4222-8222-222222222222","customerId":"11111111-1111-4111-8111-111111111111","url":"https://cliente.example/webhooks","statuses":["SHIPPED","DELIVERED"],"active":true,"secret":"exemplo-nao-utilizar"}
```
`400` validação, `401` autenticação e `403` autorização. **Hipótese:** exibir secret somente na criação e rotação; não retornar em listagens nem logs.

**Listar configurações**

Tipo: endpoint; método: `GET /api/v1/webhooks?customerId=11111111-1111-4111-8111-111111111111`. Header: `Authorization: Bearer <JWT>`. Requisição sem corpo; resposta proposta `200 OK`:
```json
{"data":[{"id":"22222222-2222-4222-8222-222222222222","customerId":"11111111-1111-4111-8111-111111111111","url":"https://cliente.example/webhooks","statuses":["SHIPPED"],"active":true}]}
```
`400` parâmetro inválido, `401` sem autenticação, `403` sem acesso. Paginação e política de autorização ainda precisam de definição.

**Editar configuração**

Tipo: endpoint; método: `PATCH /api/v1/webhooks/22222222-2222-4222-8222-222222222222`. Headers: `Authorization: Bearer <JWT>` e `Content-Type: application/json`. Exemplo:
```json
{"statuses":["DELIVERED"],"active":true}
```
Resposta proposta `200 OK`:
```json
{"id":"22222222-2222-4222-8222-222222222222","url":"https://cliente.example/webhooks","statuses":["DELIVERED"],"active":true}
```
`400` inválido, `401` não autenticado, `403` sem acesso, `404` não encontrado. Efeito de edição em entregas já pendentes requer decisão.

**Remover configuração**

Tipo: endpoint; método: `DELETE /api/v1/webhooks/22222222-2222-4222-8222-222222222222`. Header: `Authorization: Bearer <JWT>`; sem corpo. Resposta proposta `204 No Content`, sem corpo; `401`, `403` ou `404` conforme autenticação, acesso ou recurso. A política para entregas pendentes após remoção está em aberto.

**Rotacionar secret**

Tipo: endpoint; método proposto: `POST /api/v1/webhooks/22222222-2222-4222-8222-222222222222/rotate-secret`. Header: `Authorization: Bearer <JWT>`; sem corpo. Resposta proposta `200 OK`:
```json
{"secret":"nova-secret-exemplo","previousSecretValidUntil":"2026-11-01T10:00:00Z"}
```
`401`, `403` ou `404`; secret anterior mantém validade por 24 h [FDD-SEC-01].

**Consultar histórico**

Tipo: endpoint; método: `GET /api/v1/webhooks/22222222-2222-4222-8222-222222222222/deliveries`. Header: `Authorization: Bearer <JWT>`; sem corpo. Retorna até os últimos 100 envios; resposta proposta `200 OK`:
```json
{"data":[{"eventId":"33333333-3333-4333-8333-333333333333","result":"success","payload":{"event_type":"order.status_changed"},"response":{"status":200,"body":"ok"},"durationMs":120}]}
```
`401`, `403` ou `404`. **Pendente:** limites e proteção de dados no armazenamento da resposta e paginação.

**Replay da DLQ**

Tipo: endpoint; método: `POST /api/v1/admin/webhooks/dead-letter/44444444-4444-4444-8444-444444444444/replay`. Header: `Authorization: Bearer <JWT de ADMIN>`; sem corpo. Resposta proposta `202 Accepted`:
```json
{"eventId":"33333333-3333-4333-8333-333333333333","status":"pending"}
```
`401` não autenticado, `403` role insuficiente, `404` item ausente; `202` significa agendado, não entregue. Conflitos de replay simultâneo requerem definição.

**Entrega de evento ao cliente**

Tipo: chamada HTTP de saída; método `POST` à URL configurada. Headers: `Content-Type: application/json`, `X-Event-Id` UUID do evento, `X-Webhook-Id` identificador da configuração, `X-Signature` assinatura HMAC-SHA256 do corpo, `X-Timestamp` horário do envio. Representação exata da assinatura e do timestamp ainda depende de decisão [FDD-PAYLOAD-01, FDD-SEC-01].
```json
{"event_id":"33333333-3333-4333-8333-333333333333","event_type":"order.status_changed","timestamp":"2026-10-30T10:00:00Z","order_id":"55555555-5555-4555-8555-555555555555","order_number":"ORD-000001","from_status":"PROCESSING","to_status":"SHIPPED","customer_id":"11111111-1111-4111-8111-111111111111","total_cents":5000}
```
Resposta exemplo do receptor, **hipótese**: `200 OK`, corpo `{"received":true}`. Significado de outros códigos e tamanho de resposta armazenada estão pendentes. O corpo não inclui itens; detalhes podem ser consultados na API de pedidos.

---

### 6. Erros, exceções e fallback

**Matriz proposta**, não aprovada na reunião; o envelope segue `errorMiddleware`. Erros genéricos de autenticação atuais podem manter seus códigos próprios [FDD-ERR-01].

| Condição | Código proposto | HTTP / tratamento |
| --- | --- | --- |
| URL não HTTPS ou inválida | `WEBHOOK_INVALID_URL` | 400; rejeitar cadastro ou edição |
| Campos ou status inválidos | `WEBHOOK_VALIDATION_ERROR` | 400; retornar detalhes de validação |
| Configuração ou item da DLQ ausente | `WEBHOOK_NOT_FOUND` | 404 |
| Acesso ao cliente não autorizado | `WEBHOOK_FORBIDDEN` | 403; política de vínculo pendente |
| Replay incompatível com estado | `WEBHOOK_REPLAY_CONFLICT` | 409; regra de concorrência pendente |
| Payload acima de 64 KB | `WEBHOOK_PAYLOAD_TOO_LARGE` | Não enviar; destino operacional pendente |
| Timeout ou falha externa | `WEBHOOK_DELIVERY_FAILED` | Registrar falha e aplicar retry, depois DLQ; não é resposta HTTP ao pedido original |
| Falha ao inserir outbox | `WEBHOOK_OUTBOX_ERROR` | Reverter transação; mapeamento HTTP específico pendente |

Exemplo de erro proposto: `{"error":{"code":"WEBHOOK_INVALID_URL","message":"URL must use HTTPS"}}`.

**Resiliência:** timeout de 10 s; intervalos acordados de 1 min, 5 min, 30 min, 2 h e 12 h, sujeitos à definição da contagem; DLQ e replay manual. Circuit breaker não foi decidido. Não há fallback por email [FDD-RETRY-01, FDD-SCOPE-02].

**Invariantes:** nenhuma chamada externa dentro da transação; evento e mudança elegível confirmam ou revertem juntos; falha posterior de envio não reverte status; replay exige ADMIN; segredo não deve aparecer em logs [FDD-TX-01, FDD-REPLAY-01, FDD-OBS-01].

---

### 7. Observabilidade

**Métricas, hipóteses para validação:** contagem de entregas por resultado, retries e entradas na DLQ; pendências e idade da mais antiga; duração de envio e tempo entre evento e entrega. Definir instrumentos, unidade, janela e limiares antes de prometer SLO [FDD-OBS-01].

**Logs:** Pino estruturado com identificadores de evento e webhook, tentativa, resultado, duração e usuário que pediu replay. Não registrar secrets ou assinatura; evitar payload e resposta completos em logs. Histórico e proteção desses dados exigem revisão de segurança [FDD-OBS-01, FDD-REPLAY-01].

**Tracing, hipótese:** spans para gravação na outbox, processamento e chamada HTTP, correlacionados por ID de evento; propagação e taxa de amostragem dependem de escolha de infraestrutura, ainda não existente no projeto [FDD-OBS-01].

**Dashboards e alertas, hipóteses:** acompanhar backlog, idade das pendências e DLQ; alertar sobre worker indisponível e crescimento de falhas. Limiar, canal e ferramentas a definir; não confundir observabilidade operacional com dashboard visual de cliente, que está fora do escopo [FDD-OBS-01, FDD-SCOPE-02].

---

### 8. Dependências e compatibilidade

| Componente | Versão / situação atual | Observações |
| --- | --- | --- |
| Node.js | `>=20` | API e futuro worker em processos distintos |
| MySQL | 8.0 no compose | Outbox e DLQ no banco existente |
| Prisma | 5.22.0 | Client independente em cada processo |
| Express / Zod | 4.21.1 / 3.23.8 | Rotas e validação conforme padrões existentes |
| Pino | 9.5.0 | Logger existente; tracing e métricas não estão especificados |

Versões acima são as atuais do repositório, não mínimos adicionais acordados [FDD-DEPS-01]. O prefixo atual é `/api/v1`; não alterar a semântica dos endpoints de pedidos ou expor a secret em respostas que não precisem dela. Contratos novos dependem de aprovação; nenhuma compatibilidade de versões de payload foi decidida.

---

### 9. Critérios de aceite técnicos

- [ ] Mudança elegível grava status, histórico, estoque aplicável e evento atomicamente; falha na outbox reverte tudo; mudança sem assinante não cria evento [FDD-TX-01].
- [ ] Worker separado faz polling de 2 s e timeout de 10 s; falhas seguem política aprovada e DLQ, sem reverter pedido [FDD-WORK-01, FDD-RETRY-01].
- [ ] URLs HTTP são recusadas; payload acima de 64 KB não é enviado; HMAC e rotação de 24 h são validados [FDD-SEC-01, FDD-PAYLOAD-01].
- [ ] Cliente identifica evento e webhook, consegue detectar duplicatas e consultar histórico dos últimos 100 envios [FDD-PAYLOAD-01, FDD-SCOPE-01].
- [ ] Replay é restrito a ADMIN e registra autoria; testes cobrem falha, DLQ, replay e reinício do worker [FDD-REPLAY-01, FDD-WORK-01].
- [ ] Não liberar acesso público antes de aprovar e testar autorização por cliente; validar classificação de respostas, contagem de retry, ordenação com falhas, política de assinatura na rotação e condições de latência [FDD-OPEN-01, FDD-AUTH-01].
- [ ] Validar métricas, logs sem secrets e tracing conforme solução aprovada; reservar pelo menos dois dias úteis para revisão de segurança antes do deploy [FDD-OBS-01, FDD-RISK-01].

---

### 10. Riscos e mitigação

Probabilidades ainda não foram estimadas; não há dados para classificá-las como baixas, médias ou altas.

#### Acesso indevido a dados de outro cliente
- **Probabilidade:** a avaliar; **impacto:** exposição de dados ou configuração.
- **Mitigação:** aprovar vínculo usuário-cliente e testar isolamento, além de autenticação e role [FDD-AUTH-01].
- **Plano de contingência:** não disponibilizar endpoints até a política ser aprovada e validada.

#### Backlog, falhas persistentes ou ordem inesperada
- **Probabilidade:** a avaliar; **impacto:** atraso, DLQ e possível processamento fora de ordem.
- **Mitigação:** medir pendências, testar falhas e definir ordenação em retry; não presumir que polling de 2 s garante entrega em 10 s [FDD-WORK-01, FDD-OPEN-01].
- **Plano de contingência:** investigar histórico e DLQ e permitir replay por administrador; replay pode ser deduplicado pelo cliente [FDD-REPLAY-01, FDD-MODEL-01].

#### Comprometimento de secret ou envio a destino indevido
- **Probabilidade:** a avaliar; **impacto:** falsificação ou exposição de dados.
- **Mitigação:** HTTPS, secrets por endpoint e rotação; revisar HMAC e geração com Sofia. Proteção contra destinos internos e redirecionamentos é uma lacuna a avaliar com segurança, não decisão da reunião [FDD-SEC-01, FDD-RISK-01].
- **Plano de contingência:** rotacionar a secret e investigar o histórico; estratégia de revogação imediata ainda depende de decisão, pois existe janela de sobreposição de 24 h.
