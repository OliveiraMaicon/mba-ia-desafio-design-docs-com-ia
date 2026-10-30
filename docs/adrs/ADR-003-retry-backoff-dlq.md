# ADR-003: Retry com backoff e dead letter queue

**Status:** Aceita na reunião; implementação pendente.

## Contexto e problema

O endpoint de um cliente pode estar indisponível no momento da entrega. É necessário limitar as tentativas sem descartar silenciosamente um evento que não foi entregue, permitindo investigação e recuperação posterior. Diego propôs intervalos crescentes e encaminhamento para uma dead letter queue (DLQ) após o limite [09:15].

## Condutores da decisão

- Cobrir indisponibilidades temporárias, inclusive manutenções de algumas horas [Diego, 09:16].
- Evitar que um evento permaneça pendente indefinidamente quando o cliente desaparece [Diego, 09:15].
- Preservar informações para depuração e reprocessamento, mantendo limpa a leitura da outbox principal [Diego, 09:18].
- Restringir e auditar a intervenção manual [Sofia, 09:36].

## Alternativas consideradas

1. **Cinco tentativas, backoff e DLQ separada:** amplia a janela de recuperação, limita o processamento automático e permite investigar e reprocessar falhas; requer armazenamento e operação da DLQ.
2. **Três tentativas:** Bruno sugeriu um limite mais agressivo; Diego observou que poderia esgotá-lo antes do fim de uma manutenção de duas horas [09:16].
3. **Retry indefinido com backoff:** evita esgotar um limite, mas pode deixar eventos pendentes para sempre [Diego, 09:15].
4. **Marcar a falha na própria outbox:** Larissa apresentou essa alternativa à tabela separada; Diego preferiu a tabela separada para facilitar a leitura e a investigação [09:17–09:18].

## Decisão

Larissa decidiu por **cinco tentativas**, com backoff descrito na reunião como exponencial e intervalos de **1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas** [09:17]. Após o limite, o evento segue para uma DLQ persistida em tabela separada, decisão confirmada no resumo [Larissa, 09:48]. Diego propôs registrar payload, motivo da falha e timestamp nessa tabela [09:18].

O replay será manual por endpoint administrativo, recolocando o evento na outbox como pendente [Diego, 09:18; Larissa, 09:19]. Seu acesso exige role ADMIN e o registro de quem realizou o replay para auditoria [Sofia, 09:36]. Cliente que não responde em 10 segundos é tratado como falha para retry [Diego e Sofia, 09:42].

**Contagem em aberto:** a reunião enuncia cinco tentativas e cinco intervalos, mas não esclarece se a chamada inicial está incluída nas cinco nem como cada intervalo se associa às chamadas. Não se deve inferir uma sequência ou um total de chamadas antes de esclarecer essa ambiguidade.

## Consequências

- Indisponibilidades temporárias têm novas oportunidades de entrega sem retry infinito; eventos que esgotarem o limite precisarão de intervenção para replay [09:15–09:19].
- A tabela separada mantém dados úteis para diagnóstico e recuperação, ao custo de armazenamento e manutenção adicionais [09:18].
- Replay requer controle de acesso e auditoria; a operação não é destinada a operadores sem role ADMIN [Sofia, 09:36].
- Permanecem a esclarecer quais outras respostas e falhas são retentáveis e como retry e replay interagem com identificação e ordenação dos eventos. Essas questões não estão resolvidas por esta decisão.

## Relações

- [ADR-001 — Outbox transacional no MySQL](ADR-001-outbox-transacional-mysql.md): registra os eventos e recebe aqueles recolocados como pendentes no replay.
- [ADR-002 — Worker separado com polling da outbox](ADR-002-worker-separado-polling.md): trata do processamento assíncrono; a interação entre retries, replay e ordenação permanece em aberto.
- [ADR-005 — At-least-once e deduplicação pelo cliente](ADR-005-at-least-once-e-deduplicacao.md): trata da possibilidade de duplicatas e dos limites de entrega associados a tentativas e replay.

## Fontes

- [`TRANSCRICAO.md`](../../TRANSCRICAO.md): Diego, Bruno, Larissa e Marcos [09:14–09:19]; Sofia [09:36]; Diego e Sofia [09:42]; resumo de Larissa [09:48].
- [Candidato ao ADR](potential-adrs/consider/WEBHOOKS/retry-backoff-dead-letter.md).
