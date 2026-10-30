# ADR-004: HMAC e rotação de segredos

- **Status:** aceita na reunião; ainda não implementada.
- **Decisora:** Sofia, com confirmação de Larissa.

## Contexto e forças

Os webhooks enviarão dados de pedidos a sistemas fora da infraestrutura do OMS. O consumidor precisa verificar a origem e a integridade do corpo recebido (Sofia, [09:19]). Um segredo global ampliaria o impacto de um vazamento; a rotação precisa dar tempo para o cliente atualizar seus sistemas (Sofia, [09:21]).

## Decisão

Assinar o corpo da requisição com **HMAC-SHA256**, usando um segredo compartilhado único por endpoint. Enviar a assinatura no header `X-Signature`, para verificação pelo cliente. A configuração do webhook deverá armazenar o segredo, e o cliente poderá solicitar um novo por um endpoint da API. Após a rotação, o segredo antigo permanecerá válido em paralelo por **24 horas**, deixando de valer depois desse período (Sofia, [09:20–09:22]; Larissa, [09:48]).

Como requisito complementar, a URL cadastrada deve usar HTTPS; URLs HTTP serão recusadas na validação (Sofia, [09:23]). A decisão é para a funcionalidade proposta, não uma descrição de comportamento já disponível no OMS.

## Alternativas consideradas

- **Segredo global:** discutido e descartado; seu vazamento afetaria todos os endpoints, ao contrário do isolamento por endpoint (Sofia, [09:21]).
- **Rotação sem sobreposição:** alternativa plausível, não registrada como discussão na reunião. Reduziria o tempo de validade do segredo antigo, mas exigiria uma troca coordenada e poderia interromper verificações durante a migração.
- **Apenas HTTPS, sem assinatura:** alternativa plausível, não registrada como discussão. Protege o transporte, mas não substitui o mecanismo escolhido para o consumidor verificar a origem e a integridade do corpo.

## Consequências

- O cliente precisa guardar o segredo e verificar a assinatura; o OMS precisa gerir segredos por endpoint e seu ciclo de vida.
- O vazamento de um segredo fica limitado ao endpoint correspondente, mas a sobreposição mantém o segredo antigo utilizável por 24 horas, equilibrando risco e continuidade da integração.
- Sofia solicitou pelo menos dois dias úteis para revisar o código de segurança, incluindo HMAC e geração de segredos, antes do deploy ([09:46]).

## Questões em aberto

- Quais bytes exatos do corpo são assinados e qual a codificação e o formato completo de `X-Signature`?
- Durante a sobreposição, qual segredo assina cada envio e como o cliente seleciona ou verifica os segredos válidos? Como lidar com rotações sucessivas?
- Como gerar, armazenar e disponibilizar os segredos com segurança, e quem pode solicitar a rotação?
- Diego propôs `X-Timestamp` para o cliente detectar replay, se desejar ([09:44]). A decisão assina o corpo, não define que o timestamp esteja autenticado; portanto, seu uso não constitui por si só proteção autenticada contra replay. A estratégia de prevenção permanece em aberto.

## Relações

- [ADR-001 — Outbox transacional no MySQL](ADR-001-outbox-transacional-mysql.md): define o snapshot do payload; os bytes exatos do corpo assinados permanecem em aberto.
- [ADR-006 — Reuso dos padrões existentes em webhooks](ADR-006-reuso-de-padroes-existentes.md): aborda a organização e validação do módulo que deverá aplicar a exigência de HTTPS.

## Fontes

- `TRANSCRICAO.md`: Sofia [09:19–09:23], Diego [09:44], Sofia [09:46] e Larissa [09:48].
- `docs/adrs/potential-adrs/consider/WEBHOOKS/hmac-rotacao-segredo.md`: levantamento da decisão e das questões em aberto.
