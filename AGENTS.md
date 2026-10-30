# Contexto do repositório

- Este é um desafio de documentação de webhooks para um OMS existente, não de implementação. Para a entrega, não altere `src/`, `prisma/`, `tests/`, configurações ou `TRANSCRICAO.md`; use-os como fontes. O enunciado e os critérios completos estão no `README.md`.
- Não invente requisitos ou decisões: rastreie cada item à transcrição (timestamp e falante) ou a um caminho real do código em `docs/TRACKER.md`. Diferencie decisões fechadas, questões em aberto e itens descartados ou adiados.
- Mantenha as alturas distintas: PRD (produto), RFC (proposta e alternativas), ADRs (decisões), FDD (implementação), tracker (origem). O enunciado exige 5–8 ADRs `docs/adrs/ADR-NNN-titulo-em-kebab-case.md`; esse formato prevalece sobre o exemplo divergente em `docs/adrs/README.md`.
- A entrega pede substituir o `README.md` pelo relato do processo; preserve acesso ao enunciado e seus critérios ao planejar essa substituição.

## Código de referência

- `src/server.ts` inicia a aplicação; `src/app.ts` conecta repositórios, serviços e controllers, e `src/routes/index.ts` monta as rotas em `/api/v1`.
- A máquina de estados está em `src/modules/orders/order.status.ts`; `OrderService.changeStatus` em `src/modules/orders/order.service.ts` atualiza estoque, status e histórico na mesma transação Prisma. O código atual não implementa webhooks, filas ou eventos externos.

## Comandos e cuidados

- Node >= 20; instale dependências com `npm ci`. `npm run build` compila apenas `src/`; `npm run lint` verifica TypeScript. `npm run format` escreve no repositório inteiro, inclusive arquivos fora da entrega documental.
- Para executar a API: copie `.env.example` para `.env`, suba o MySQL com `docker compose up -d`, aplique migrações com `npm run db:migrate` e rode `npm run dev`. `npm run db:seed` é opcional.
- Os testes são de integração com MySQL: `tests/setup.ts` apaga dados de pedidos, produtos, clientes e usuários **antes de cada teste**. Use um banco de teste isolado e migrado, nunca um banco com dados a preservar; configure `DATABASE_URL` e `JWT_SECRET` no ambiente. Execute `npm test` ou, para um arquivo, `npm test -- tests/orders.test.ts`. Não use `npm run db:reset` em banco com dados a preservar.
