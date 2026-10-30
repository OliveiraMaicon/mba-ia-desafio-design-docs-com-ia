# ADR-005: At-least-once e deduplicação pelo cliente

- **Status:** Aceito na reunião; implementação pendente.

## Contexto e problema

Clientes precisam ser notificados sobre mudanças de status de pedidos. Uma tentativa de entrega pode falhar mesmo que o cliente já tenha recebido o evento, por exemplo, se não responder antes do timeout. Uma nova tentativa pode, portanto, produzir uma duplicata. É necessário definir o contrato de entrega e quem evita efeitos repetidos no processamento.

## Direcionadores

- Evitar a complexidade de coordenação entre plataforma e cliente exigida por exactly-once.
- Permitir recuperação de falhas temporárias sem esconder a possibilidade de duplicatas.
- Tornar a responsabilidade do consumidor explícita no contrato externo.

## Opções consideradas

1. **At-least-once com deduplicação pelo cliente:** usar um identificador por evento e admitir entregas repetidas.
2. **Exactly-once:** buscar processamento exatamente uma vez com coordenação entre emissor e consumidor; descartado pela complexidade apontada na reunião.

## Decisão

Adotar o contrato denominado **at-least-once**, com possibilidade de recebimento duplicado. Um UUID é gerado para cada evento ao entrar na outbox e enviado no header `X-Event-Id`; o cliente é responsável por deduplicar pelo identificador e tratar seu processamento de forma idempotente. O contrato deve ser destacado na documentação para desenvolvedores.

O termo não é uma garantia incondicional de entrega efetiva: após o limite de tentativas, o evento vai para a DLQ, e sua recuperação depende de ação operacional, como o replay manual previsto. Se não houver recuperação bem-sucedida, o cliente pode não receber o evento. A outbox transacional registra o evento junto à mudança de status, mas não assegura que o endpoint remoto o receba.

## Consequências

- **Positivas:** retentativas podem superar indisponibilidades temporárias sem exigir a coordenação de exactly-once; o identificador permite ao consumidor reconhecer duplicatas.
- **Negativas:** o consumidor precisa armazenar e consultar identificadores e evitar efeitos duplicados; o sistema precisa comunicar falhas e manter a operação da DLQ. O contrato, por si só, não garante processamento exatamente uma vez.
- **Em aberto:** a reunião não especifica como o identificador se comporta em todas as tentativas e no replay de DLQ, nem o prazo de retenção de identificadores pelo cliente. Também não define a identidade do evento quando há múltiplos endpoints; não se presume uma decisão de fan-out. Esses detalhes precisam ser resolvidos antes de se especificar o algoritmo de deduplicação.

## Relações

- [ADR-001 — Outbox transacional no MySQL](ADR-001-outbox-transacional-mysql.md): registra o evento com UUID, sem garantir sua entrega ao endpoint.
- [ADR-003 — Retry com backoff e dead letter queue](ADR-003-retry-backoff-dlq.md): define tentativas limitadas e replay, que se relacionam à possibilidade de duplicatas e aos limites do contrato.

## Fontes

- [Transcrição](../../TRANSCRICAO.md): Diego e Larissa [09:24–09:26] decidem o contrato e o UUID no header; Marcos [09:26] assume a documentação; Larissa [09:48] confirma no resumo.
- [Transcrição](../../TRANSCRICAO.md): Diego e Larissa [09:15–09:19] definem limite de tentativas, DLQ e replay manual; Diego [09:42] define timeout como falha sujeita a retry; Diego e Bruno [09:06–09:08, 09:40–09:41] descrevem a outbox transacional.
- [Candidato ao ADR](potential-adrs/consider/WEBHOOKS/at-least-once-deduplicacao.md): análise e questões não resolvidas.
