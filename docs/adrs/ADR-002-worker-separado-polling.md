# ADR-002: Worker separado com polling da outbox

**Status:** Aceita na reunião técnica; ainda não implementada.

## Contexto e problema

As mudanças de status precisam gerar notificações sem atrelar a transação de pedidos à disponibilidade ou à latência dos endpoints dos clientes. A reunião escolheu registrar eventos em uma outbox no MySQL e enviá-los de forma assíncrona ([transcrição](../../TRANSCRICAO.md), Bruno e Diego, 09:04–09:06). É preciso decidir como o processo de entrega descobre esses eventos e onde ele executa.

O código atual inicia a API em [`src/server.ts`](../../src/server.ts) e compõe seus módulos em [`src/app.ts`](../../src/app.ts); ainda não há implementação de worker ou de webhooks. A [RFC](../RFC.md) descreve a solução como proposta em revisão, não como capacidade existente.

## Condutores da decisão

- Desacoplar chamadas HTTP externas da transação de pedidos (Bruno e Diego, 09:04–09:06).
- Reutilizar o MySQL e a stack existentes sem introduzir infraestrutura adicional para um time pequeno (Larissa e Diego, 09:07).
- Atender à expectativa de notificações em menos de 10 segundos, ainda sem definição de medição ponta a ponta (Diego e Marcos, 09:09–09:10; RFC-OPEN-03).
- Limitar o efeito de reinícios da API sobre o processo de entrega (Diego, 09:11).

## Alternativas consideradas

1. **Worker separado consultando a outbox por polling:** consulta periódica ao banco existente, com custo de operação e latência de espera.
2. **Trigger MySQL para acordar o worker:** Bruno a sugeriu; Diego observou que a trigger executa SQL, mas não notifica diretamente um processo externo, exigindo um mecanismo adicional (09:09).
3. **Worker na mesma instância da API:** Diego apontou que um reinício da API também interromperia o worker (09:11).
4. **Redis Streams ou infraestrutura semelhante:** alternativa discutida para a entrega assíncrona; adicionaria infraestrutura em vez de reutilizar o MySQL (Larissa e Diego, 09:07).

## Decisão

Executar **um único worker em processo separado da API**, consultando a outbox por polling a cada **2 segundos**. Ele busca os eventos pendentes mais antigos, processa-os e registra o resultado. A escolha do polling foi registrada por Larissa (09:10), a separação de processo exigida por Diego (09:11) e ambas foram confirmadas no resumo da reunião (Larissa, 09:48).

O worker usará o mesmo banco e a mesma stack da API, mas terá uma instância própria de `PrismaClient`, por ser outro processo (Diego e Bruno, 09:11 e 09:29–09:30). Foi proposta uma entrada separada `src/worker.ts`, com a lógica no módulo de webhooks; esses caminhos são proposta, não arquivos existentes (Larissa, 09:11; Bruno e Diego, 09:28).

Na reunião, Diego associou a leitura por `created_at` com worker único à ordenação por pedido; Larissa limitou essa intenção a `order_id` enquanto houver um único worker, sem garantia de ordem global (09:12–09:13). **A interação com retries e replay não foi resolvida**: não se deve prometer que a ordem de entrega por pedido está efetivamente garantida apenas pelo uso de um worker (RFC-OPEN-05).

## Consequências

### Positivas

- A entrega HTTP fica fora do caminho transacional de pedidos, e o processo de entrega não depende do ciclo de vida da instância da API.
- O MySQL existente é reutilizado, sem uma infraestrutura de mensageria adicional para esta decisão.

### Negativas e riscos

- O polling gera consultas periódicas e espera variável; o intervalo de 2 segundos, por si só, **não garante entrega ponta a ponta em menos de 10 segundos**. Backlog, lentidão do cliente, falhas e retries podem aumentar a latência (RFC-OPEN-03).
- Um processo adicional precisa ser iniciado e acompanhado. Se parar ou não acompanhar a demanda, eventos podem acumular na outbox.
- A abordagem com worker único limita a concorrência; múltiplos workers podem invalidar a intenção de ordenação por pedido. Particionamento por `order_id` ou lock pessimista foram citados como possibilidades futuras, não escolhidas (Diego, 09:13).

## Questões em aberto

- Como iniciar, supervisionar e recuperar o worker e tratar backlog ou parada?
- Como definir e medir a expectativa de menos de 10 segundos em condições reais de carga e disponibilidade (RFC-OPEN-03)?
- Como tratar a ordem por pedido quando uma entrega falha e outra está pronta, inclusive após replay (RFC-OPEN-05)?

## Relações

- [ADR-001 — Outbox transacional no MySQL](ADR-001-outbox-transacional-mysql.md): registra os eventos que o worker consulta.
- [ADR-003 — Retry com backoff e dead letter queue](ADR-003-retry-backoff-dlq.md): trata das falhas e do replay; sua interação com a ordenação permanece em aberto.
- [ADR-006 — Reuso dos padrões existentes em webhooks](ADR-006-reuso-de-padroes-existentes.md): descreve a organização proposta para o worker e sua instância própria de PrismaClient.

## Fontes

- [Transcrição da reunião](../../TRANSCRICAO.md): Bruno e Diego, 09:04–09:06; Larissa e Diego, 09:07–09:13; Bruno e Diego, 09:28–09:30; Larissa, 09:48.
- [RFC — Webhooks de notificação de pedidos](../RFC.md): proposta técnica, alternativas, riscos e questões RFC-OPEN-03 e RFC-OPEN-05.
- [Candidato ao ADR](potential-adrs/consider/WEBHOOKS/worker-separado-polling.md).
