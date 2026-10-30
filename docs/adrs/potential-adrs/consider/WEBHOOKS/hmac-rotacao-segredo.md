# Potencial ADR: HMAC-SHA256 e rotação de segredo

**Módulo:** WEBHOOKS / AUTH
**Categoria:** segurança
**Prioridade:** consider — 60/150 (0 base + 25 impacto + 20 custo + 15 conhecimento); exceção ao limiar, conforme pedido
**Identificado em:** 2026-10-30

## O que foi identificado

Sofia decide HMAC-SHA256 sobre o corpo, segredo único por endpoint e rotação com validade simultânea por 24 horas [09:20–09:22], confirmado por Larissa [09:48]. A assinatura permite ao cliente validar origem e integridade; HTTPS obrigatório foi exigido [09:23].

## Por que documentar

- **Impacto:** contrato de segurança compartilhado com todos os consumidores.
- **Trade-off:** segredo por endpoint limita impacto de vazamento, mas acrescenta ciclo de vida e período de transição.
- **Conhecimento:** implementação e integração exigem clareza sobre bytes assinados, disponibilização e rotação; Sofia pede revisão de segurança antes do deploy [09:46–09:49].

## Evidências

- `TRANSCRICAO.md`: Sofia [09:20–09:22] descreve assinatura e rotação; Diego [09:44] inclui `X-Signature` e `X-Timestamp`. A reunião não define algoritmo de seleção de segredo ou formato completo da assinatura durante a janela.
- `src/modules/auth/` trata autenticação existente e `src/middlewares/auth.middleware.ts` autorização; não há implementação de assinatura outbound. `prisma/schema.prisma` não contém segredos de webhook.
- A base em `7ef4317` (2026-06-24, `init repository`) antecede implementação da feature e não informa o motivo da escolha criptográfica.

## Questões para o ADR

- Como os clientes verificam assinatura durante a sobreposição de segredos e qual formato exato do header?
- Como armazenar e expor segredos com segurança, e quem pode rotacioná-los?
- O timestamp está fora do corpo assinado segundo a reunião: como tratar replay attack sem presumir que está autenticado?

## Candidatos relacionados

[At least once](at-least-once-deduplicacao.md) · [Padrões existentes](reuso-padroes-existentes.md)
