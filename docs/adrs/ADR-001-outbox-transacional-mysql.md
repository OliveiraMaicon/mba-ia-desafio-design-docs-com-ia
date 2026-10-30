# ADR-001 — Outbox transacional no MySQL

**Status:** Aceita na reunião técnica; implementação ainda inexistente.

## Contexto e problema

Clientes B2B precisam ser notificados sobre mudanças de status dos pedidos, sem depender de consultas periódicas. O OMS já executa a mudança de status, o histórico e, quando aplicável, a alteração de estoque em uma transação. Uma chamada HTTP dentro dela vincularia a operação à latência e à disponibilidade do destinatário. Precisamos registrar a intenção de notificar sem perder o evento quando a mudança for confirmada, nem registrar um evento para uma mudança revertida (Marcos [09:02]; Bruno e Larissa [09:03–09:04]; Diego [09:06]).

O MySQL já é o banco do projeto. No código atual, `OrderService.changeStatus` utiliza uma transação Prisma para estoque, pedido e histórico; não há inserção em outbox nem implementação de webhooks. A RFC descreve a solução como proposta para essa base existente.

## Decisão

Adotar uma **outbox no MySQL existente** para eventos elegíveis de mudança de status. A inserção deve ocorrer na mesma transação SQL que atualiza o pedido, seu histórico e o estoque quando aplicável. Se a inserção falhar, a mudança deve ser revertida; se a transação for revertida, o evento também não deve persistir. Um worker separado lerá a outbox e fará as chamadas HTTP fora da transação (Diego [09:06]; Larissa [09:08]; Bruno e Diego [09:40–09:41]).

Antes de inserir, verificar os status de interesse dos webhooks do cliente: sem interessado no status, nenhum evento é inserido (Marcos [09:33]; Bruno e Diego [09:34]). O evento terá identificador UUID e guardará o payload já renderizado como snapshot do momento da mudança, em vez de reconstruí-lo a partir do pedido no envio (Larissa e Diego [09:51–09:52]).

Essa atomicidade garante o registro do evento elegível junto à mudança, **não a entrega HTTP bem-sucedida**. O processamento posterior e suas falhas são tratados separadamente na RFC.

## Alternativas consideradas

### Envio HTTP síncrono durante a mudança de status

Descartado: um destinatário lento prolongaria a transação e sua indisponibilidade afetaria o fluxo de pedidos. A outbox evita a chamada externa na transação, ao custo de processamento assíncrono e operação adicional (Bruno e Larissa [09:04]; Diego [09:06]).

### Redis Streams ou infraestrutura Redis

Considerada e não escolhida: acrescentaria infraestrutura para um time pequeno. Reutilizar o MySQL evita essa dependência, mas transfere para o banco e para o worker a responsabilidade de armazenar e processar os eventos (Larissa e Diego [09:07]).

### Guardar apenas o ID do pedido e renderizar no envio

Não escolhida: o pedido pode mudar entre a transição e a entrega, fazendo o payload refletir um estado posterior. O snapshot preserva o estado do evento no momento da inserção (Bruno [09:51]; Larissa e Diego [09:52]).

## Consequências

### Positivas

- Pedido, histórico, estoque aplicável e registro do evento elegível compartilham a fronteira de commit e rollback, evitando perda por uma gravação separada após o commit.
- A entrega externa não bloqueia a transação de pedidos; a solução aproveita o MySQL existente sem exigir Redis.
- Filtrar antes da inserção reduz registros desnecessários; o snapshot evita que alterações posteriores do pedido modifiquem o conteúdo do evento.

### Negativas e riscos

- Uma falha na gravação da outbox impede a mudança de status, mesmo que o pedido pudesse ser atualizado isoladamente.
- A tabela pode crescer e sua leitura requer cuidado com desempenho. Diego mencionou índices, lotes pequenos e arquivamento de linhas entregues após aproximadamente 30 dias; o arquivamento ficou fora do escopo desta feature [09:08], não é uma capacidade implementada.
- O worker precisa ser operado; o registro atômico não garante entrega imediata nem sucesso do destinatário. A retenção de snapshots também aumenta os dados persistidos.

## Relações

- [ADR-002 — Worker separado com polling da outbox](ADR-002-worker-separado-polling.md): trata da leitura e entrega assíncrona dos eventos registrados na outbox.
- [ADR-003 — Retry com backoff e dead letter queue](ADR-003-retry-backoff-dlq.md): trata de falhas de entrega e replay para a outbox.
- [ADR-004 — HMAC e rotação de segredos](ADR-004-hmac-e-rotacao-de-segredos.md): trata da assinatura do corpo enviado; os bytes exatos assinados permanecem em aberto.
- [ADR-005 — At-least-once e deduplicação pelo cliente](ADR-005-at-least-once-e-deduplicacao.md): trata do identificador do evento e dos limites da garantia de entrega.

## Fontes

- [Transcrição](../../TRANSCRICAO.md): Marcos [09:02, 09:33]; Larissa [09:03–09:04, 09:07–09:08, 09:51–09:52]; Bruno [09:04, 09:34, 09:40–09:41, 09:51]; Diego [09:06–09:08, 09:34, 09:41, 09:52].
- [RFC — proposta, alternativas e riscos](../RFC.md): fluxo e limites, alternativas consideradas e impactos.
- [`src/modules/orders/order.service.ts`](../../src/modules/orders/order.service.ts): `changeStatus` e sua transação existente; não insere evento.
- [`prisma/schema.prisma`](../../prisma/schema.prisma): datasource MySQL, modelos de pedido e histórico existentes; não há modelo de outbox.
- [Candidato a ADR](potential-adrs/must-document/WEBHOOKS/outbox-transacional-mysql.md).
