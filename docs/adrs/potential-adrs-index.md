# Índice de candidatos a ADR — webhooks

## Progresso

- **WEBHOOKS**: analisado em 2026-10-30, com ORDERS, AUTH e PLATFORM como dependências; 1 alta prioridade, 1 pontuação média e 4 exceções solicitadas.
- Outros módulos: mapeados para contexto, fora do escopo de identificação de decisões genéricas.
- `docs/adrs/generated/` não existe; `docs/adrs/README.md` é orientação, não ADR formal. Não há duplicata formal identificada.

## Critérios e ressalva

As pontuações são estimativas de impacto, custo de mudança e conhecimento, não fatos da reunião. O método do agente atribui base 75 apenas às categorias estruturais do passo 0; decisões comuns partem de zero e têm máximo 75. A outbox, como infraestrutura crítica para esta feature, recebe base 75. As demais seguem o cálculo literal e, quando abaixo de 75, ficam em `consider/` **como exceção explícita ao corte**, para preservar os seis candidatos expressamente solicitados. Sua presença não indica aprovação de ADR formal nem cumprimento do limiar padrão. Em todos os casos, a decisão da reunião não está implementada atualmente.

## Alta prioridade — must-document

| Candidato | Categoria | Pontuação (base + impacto + custo + conhecimento) |
| --- | --- | --- |
| [Outbox transacional no MySQL](potential-adrs/must-document/WEBHOOKS/outbox-transacional-mysql.md) | Infraestrutura crítica | 120 (75+20+15+10) |

## Para consideração — consider

| Candidato | Categoria | Pontuação | Classificação |
| --- | --- | --- | --- |
| [Worker separado por polling](potential-adrs/consider/WEBHOOKS/worker-separado-polling.md) | Execução | 50 (0+20+15+15) | Exceção solicitada |
| [Retry e dead letter queue](potential-adrs/consider/WEBHOOKS/retry-backoff-dead-letter.md) | Resiliência | 50 (0+20+15+15) | Exceção solicitada |
| [HMAC e rotação de segredo](potential-adrs/consider/WEBHOOKS/hmac-rotacao-segredo.md) | Segurança | 60 (0+25+20+15) | Exceção solicitada |
| [At least once e deduplicação](potential-adrs/consider/WEBHOOKS/at-least-once-deduplicacao.md) | Contrato externo | 75 (0+25+25+25) | Limiar médio |
| [Reuso dos padrões do projeto](potential-adrs/consider/WEBHOOKS/reuso-padroes-existentes.md) | Organização | 45 (0+20+10+15) | Exceção solicitada |

## Síntese

Seis candidatos; nenhum ADR formal criado. Parâmetros como polling de 2 s, progressão de retry e janela de rotação foram consolidados nos candidatos respectivos, e não tratados como decisões isoladas. Decisões da reunião foram verificadas contra a transcrição; o histórico disponível só registra a base inicial, não o motivo das escolhas.
