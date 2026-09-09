# 12. Protocolos e formatos de mercado

Regra: onde existe um protocolo ou formato aberto adotado pelo mercado, o
produto o usa sem modificacao. Formato proprio so onde nao ha padrao, e
sempre com um adaptador para o formato de fato mais usado.

## Mapa

| Area | Padrao adotado | Mantido por | Situacao |
|------|----------------|-------------|----------|
| Ferramentas, recursos e prompts | MCP (Model Context Protocol) | Agentic AI Foundation, Linux Foundation | aberto |
| Skills | Agent Skills (`SKILL.md`) | especificacao aberta publicada pela Anthropic | aberto, adotado por varios clientes |
| Agente falando com agente | A2A (Agent2Agent) | Linux Foundation | aberto |
| Schema de argumentos de ferramenta | JSON Schema | IETF e comunidade | aberto |
| Autenticacao de servidor remoto | OAuth 2.1 com PKCE | IETF | aberto, exigido pelo MCP |
| Webhooks de entrada e saida | Standard Webhooks | comunidade (Svix) | aberto |
| Assinatura de webhook por origem | cabecalhos nativos de GitLab, GitHub, Jira | cada plataforma | de fato |
| Agendamento | expressao cron, com fuso IANA | POSIX | aberto |
| Telemetria e custo | OpenTelemetry, convencoes semanticas de GenAI | CNCF | aberto |
| Perfis de agente | Markdown com frontmatter, campos do Claude Code | de fato | adaptador |
| Plugins | layout de plugin do Claude Code | de fato | adaptador |
| Hooks | formato proprio com adaptador para hooks do Claude Code | nao ha padrao | adaptador |
| Canais | API oficial de cada plataforma (Telegram Bot API, Slack Events API) | cada plataforma | de fato |
| Transporte cliente e daemon | WebSocket com quadros JSON, formato proprio | nao ha padrao para este caso | proprio |

## Como cada um entra

### MCP

Ja e a base de ferramentas. O daemon e um cliente MCP completo: stdio e
HTTP streamable, tools, resources, prompts, notificacoes de mudanca,
OAuth 2.1 para servidores remotos. Nada de extensao proprietaria por cima
do protocolo.

O daemon tambem **expoe** um servidor MCP. Qualquer cliente MCP de
mercado, incluindo o Claude Desktop e o Claude Code, pode listar os
agentes como ferramentas (`run_agent`, `list_sessions`, `cost_report`) e
disparar trabalho na maquina. Isso substitui integracoes ponto a ponto.

### Agent Skills

O formato de skill em `09-extensoes.md` e o da especificacao Agent
Skills: pasta com `SKILL.md`, frontmatter com `name` e `description`,
arquivos auxiliares ao lado. O campo `activate` de `11-determinismo.md` e
uma extensao nossa; skills sem ele continuam validas e clientes que nao o
entendem o ignoram, como a especificacao preve para campos desconhecidos.

### A2A

Duas direcoes:

- **Expor.** O daemon publica um Agent Card em `/.well-known/agent.json`
  (via relay ou VPN) e aceita tarefas A2A. Um agente de terceiro, ou outro
  daemon seu, delega trabalho por um protocolo que ele ja fala, com
  autenticacao e streaming de estado padronizados. E o caminho de mercado
  para "terceiro dispara acao", ao lado dos webhooks.
- **Consumir.** Um perfil pode listar agentes A2A externos em
  `delegates` como se fossem perfis locais. O harness trata a delegacao
  igual, com custo registrado quando o agente remoto informa uso.

A2A entra na fase 4. Webhooks entram antes porque GitLab, GitHub e Jira
falam webhook, nao A2A.

### JSON Schema

Toda ferramenta nativa, MCP ou de workflow descreve argumentos em JSON
Schema. A validacao de `11-determinismo.md` usa o mesmo schema que vai ao
provedor. Um unico validador, sem formato intermediario.

### Standard Webhooks

Webhooks de saida (`run.end`, `budget.exceeded`, `approval.required`)
seguem Standard Webhooks: cabecalhos `webhook-id`, `webhook-timestamp` e
`webhook-signature` com HMAC SHA-256. Gatilhos de entrada `generic`
validam com a mesma regra. Gatilhos de fonte conhecida validam com o
mecanismo nativo da fonte (`X-Gitlab-Token`, `X-Hub-Signature-256`,
assinatura JWT do Jira Connect).

### Cron

Expressao cron de cinco campos, com fuso horario IANA obrigatorio. Sem
sintaxe propria. Biblioteca de parsing de mercado, nao implementacao
propria.

### OpenTelemetry

O ledger e a fonte de custo, mas cada chamada ao provedor tambem emite um
span com os atributos das convencoes semanticas de GenAI
(`gen_ai.system`, `gen_ai.request.model`, `gen_ai.usage.input_tokens`,
`gen_ai.usage.output_tokens`). Exportador OTLP desligado por padrao.
Quem ja tem Grafana, Datadog ou Langfuse liga e ve o produto ao lado do
resto.

### Formatos de fato do Claude Code

Perfis, plugins e hooks nao tem padrao aberto. O produto le o formato do
Claude Code por ser o mais usado e o que o usuario ja tem, com as
adaptacoes de `09-extensoes.md`. Se surgir um padrao aberto para esses
tres, o adaptador muda e o formato interno nao.

## O que fica proprio e por que

- **Protocolo cliente e daemon.** Nao ha padrao para "interface remota de
  um harness pessoal". Fica JSON sobre WebSocket, documentado em
  `02-arquitetura.md`, com versao no quadro `auth` para evoluir.
- **Perfil de agente.** O frontmatter usa os campos do Claude Code onde
  existem (`name`, `description`, `tools`, `model`) e acrescenta os
  nossos (`provider`, `budget`, `phases`, `context`). Um perfil nosso
  continua carregavel pelo Claude Code, que ignora o que nao conhece.
- **Workflows.** Nao ha padrao maduro para workflow de agente com custo
  calculavel. Fica YAML proprio, pequeno, documentado em
  `11-determinismo.md`.

## Consequencias para o roadmap

- Fase 1 ja nasce com MCP completo, Agent Skills e JSON Schema.
- Fase 3 traz webhooks no formato Standard Webhooks e cron.
- Fase 4 traz o servidor MCP do daemon, A2A nas duas direcoes e OTel.
