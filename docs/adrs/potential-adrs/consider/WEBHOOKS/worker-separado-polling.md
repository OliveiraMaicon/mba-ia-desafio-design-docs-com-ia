# Potencial ADR: Worker separado com polling

**Módulo:** WEBHOOKS / PLATFORM
**Categoria:** execução
**Prioridade:** consider — 50/150 (0 base + 20 impacto + 15 custo + 15 conhecimento); exceção ao limiar, conforme pedido
**Identificado em:** 2026-10-30

## O que foi identificado

Diego propõe polling a cada 2 segundos [09:09], Larissa registra a decisão [09:10] e Diego exige processo separado da API [09:11]. O resumo de Larissa confirma ambos [09:48]. A mesma stack e banco são reutilizados, com PrismaClient por processo [09:11, 09:30].

## Por que documentar

- **Impacto:** isola entrega HTTP do caminho transacional e muda operação e implantação.
- **Trade-off:** polling evita infraestrutura de notificação externa, mas gera consultas periódicas e latência variável; 2 segundos não garante entrega ponta a ponta em menos de 10 segundos.
- **Conhecimento:** time precisa distinguir API e worker e compreender interrupções e backlog.

## Evidências

- `TRANSCRICAO.md`: Diego [09:09] contrapõe polling a trigger MySQL, que não notifica diretamente processo externo; [09:12–09:13] discute single-worker e ordenação. A ordenação por pedido foi afirmada para single-worker, mas não se demonstrou como manter ordem na presença de retries; não presumir garantia efetiva.
- `src/server.ts` e `src/app.ts` são a entrada e composição atuais da API; `src/config/database.ts` contém a configuração de acesso ao banco. Não há `src/worker.ts` no código atual.
- O commit `7ef4317` (2026-06-24, `init repository`) introduz a base; nenhuma mensagem de commit encontrada registra a decisão do worker.

## Questões para o ADR

- Como iniciar, supervisionar e recuperar o worker separado?
- Como lidar com lentidão, backlog, parada e concorrência? Que limitações de ordenação existem com retries?
- Como medir a latência real diante do intervalo de polling?

## Candidatos relacionados

[Outbox](../../must-document/WEBHOOKS/outbox-transacional-mysql.md) · [Retry e DLQ](retry-backoff-dead-letter.md)
