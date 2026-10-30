# Da reunião ao design: webhooks de pedidos

## Sobre o desafio

Este projeto transforma a transcrição de uma reunião técnica em um pacote de documentação para uma feature de webhooks de notificação de pedidos. A proposta parte de um OMS existente e descreve como avisar clientes B2B quando o status de seus pedidos mudar, sem fazer a operação de pedidos depender de uma chamada HTTP externa.

A entrega é documental: PRD, RFC, FDD, ADRs e tracker relacionam necessidades, decisões e detalhes técnicos à transcrição e ao código existente. A feature **não foi implementada**.

## Ferramentas de IA utilizadas

- **OpenCode**, com o modelo **GPT-6.1 Terra**, da OpenAI: leitura do repositório e da transcrição, elaboração dos documentos, revisão de consistência e verificações de rastreabilidade.
- **Subagentes do OpenCode**: mapeamento e identificação de decisões arquiteturais, geração dos seis ADRs e vinculação entre eles.
- **Skill `prd`**: estruturação do PRD e levantamento de dúvidas sobre pontos que a reunião não resolveu.
- **Prompts locais `fdd-prompt-generator.md` e `adr-*.prompt.md`**, do repositório `ia-resources`: condução do FDD e fluxo de mapeamento, identificação, geração e vinculação de ADRs. As saídas foram adaptadas ao formato e às restrições do desafio.

## Workflow adotado

1. O agente examinou o enunciado, a transcrição, as configurações e pontos representativos do OMS. Um `AGENTS.md` registrou as restrições da entrega e os cuidados para sessões posteriores.
2. O PRD foi produzido primeiro, usando a skill `prd`; a RFC veio em seguida, separando a proposta arquitetural de detalhes de implementação.
3. Para o FDD, foi usado o prompt local de geração e uma entrevista pontual. As propostas necessárias para detalhar a implementação foram marcadas como hipóteses quando não havia decisão na reunião.
4. Os prompts de ADR foram executados na sequência **mapear, identificar, gerar e relacionar**. Os seis ADRs finais foram numerados e colocados diretamente em `docs/adrs/`, conforme o enunciado; os artefatos intermediários foram mantidos para consulta. A RFC recebeu links para os ADRs depois de criados.
5. O tracker foi atualizado ao longo da produção e depois auditado contra os documentos, a transcrição e os caminhos do código. A verificação automatizada conferiu IDs, arquivos e pares de timestamp e falante, mas não mediu formalmente a porcentagem de cobertura dos itens identificáveis.

## Prompts customizados

Os trechos abaixo são **excertos de solicitações feitas nesta sessão**, reduzidos para destacar as instruções de trabalho.

### PRD com a skill

```text
Rode a skill @.agents/skills/prd/
Leia o @TRANSCRICAO.md

Produza o arquivo docs/PRD.md cobrindo a feature de Sistema de Webhooks
de Notificação de Pedidos.
```

O pedido também listou as seções obrigatórias e exigiu itens explicitamente descartados ou adiados na seção de fora de escopo.

### FDD com prompt local

```text
Execute o prompt
/Users/oliveiramaicon/workspace/ia-resources/prompts/fdd-prompt-generator.md

Produza o arquivo docs/FDD.md detalhando o "como implementar" da feature.
```

O contexto adicional solicitou fluxos, contratos, erros, observabilidade, critérios de aceite e integração com pelo menos quatro arquivos reais do sistema existente. O caminho acima é local ao ambiente usado na produção; não é uma dependência necessária para ler a entrega.

### ADRs em sequência

```text
Rode os prompts abaixo para gerar os ADRs.
Se achar necessário rode com subagentes
/Users/oliveiramaicon/workspace/ia-resources/prompts/adr-*

acredito que a order seja adr-map, adr-identify, adr-generate e adr-link,
mas valide para confirmar.
```

## Iterações e ajustes

Foram **três ajustes principais documentados nesta sessão**, além da produção inicial dos arquivos. Não há registro de uma contagem maior de ciclos de revisão humana, e as verificações abaixo não equivalem à aprovação técnica da solução.

1. **Ambiguidades de retry e latência:** a reunião cita cinco tentativas e cinco intervalos sem definir se a primeira chamada entra na contagem; também não estabelece como medir a meta de menos de 10 segundos. Em vez de assumir uma resposta, a interação confirmou que ambos deveriam permanecer em aberto no PRD e nos documentos seguintes.
2. **FDD e decisões não tomadas:** faltavam, entre outros pontos, autorização por cliente e identidade do evento em múltiplos endpoints e replay. O agente apresentou opções; o usuário escolheu registrar propostas sujeitas a validação. O FDD distingue essas hipóteses das decisões da reunião e sinaliza bloqueios antes da disponibilização.
3. **ADRs e rastreabilidade:** os prompts de ADR produzem saídas intermediárias e usam critérios de pontuação que deixaram parte das decisões específicas da feature abaixo do corte. O fluxo foi adaptado para documentar as seis decisões exigidas no formato do desafio. Depois, uma auditoria ampliou o tracker e esclareceu que fontes relacionadas a hipóteses dão contexto, não aprovação dos detalhes propostos.

## Como navegar a entrega

Ordem sugerida:

1. [PRD](docs/PRD.md): problema, público, objetivos, requisitos e exclusões.
2. [RFC](docs/RFC.md): proposta arquitetural, alternativas e questões em revisão.
3. [Índice dos ADRs](docs/adrs/README.md): seis decisões arquiteturais, suas alternativas e consequências.
4. [FDD](docs/FDD.md): fluxos, contratos propostos, erros, integração com o OMS e critérios técnicos.
5. [Tracker](docs/TRACKER.md): origem de decisões, requisitos e pontos em aberto na [transcrição](TRANSCRICAO.md) ou no código.

O [mapeamento](docs/adrs/mapping.md) e o [índice de candidatos a ADR](docs/adrs/potential-adrs-index.md) registram as etapas intermediárias de análise.
