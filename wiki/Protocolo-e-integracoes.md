# Protocolo e integracoes

## Protocolo WebSocket

Clientes falam com o daemon em `ws://127.0.0.1:47311/ws` com quadros JSON
tipados em `@agent-hub/core` (`protocol/frames.ts`).

- O primeiro quadro autentica e informa `protocol_version`.
- Pedidos tem resposta com tipo proprio: `session.create` responde
  `session.created`, `run.start` responde `run.started`, e assim por diante.
- Eventos de run chegam como `event` com `seq` por sessao. Depois de cair, o
  cliente manda `sync` com o ultimo `seq` e recebe o que perdeu.

Grupos de quadros: sessoes, runs, aprovacoes, orcamento, custos, agentes e papeis,
agendamentos e automacao, gatilhos, conectores e OAuth, chaves, contexto do
projeto, workspace e arquivos, terminais, tarefas em segundo plano, autenticacao
e dispositivos, saude e controle do daemon.

Eventos de run: texto e raciocinio em streaming, chamada e resultado de
ferramenta, uso e custo por passo, roteamento, prompt melhorado, contexto e
ferramentas selecionadas, compactacao, delegacao, fase, gancho, verificacao,
escalada e fim de run.

`NodeDaemonClient`, no core, e um cliente pronto para Node, direto ou pelo relay.

## Servidor MCP

`agent-hub-daemon mcp` expoe o daemon por stdio para o Claude Code, o Claude
Desktop ou outro cliente MCP, com `list_agents`, `list_sessions`, `run_agent` e
`cost_report`.

```json
{
  "mcpServers": {
    "agent-hub": {
      "command": "node",
      "args": ["/caminho/agent-hub/daemon/dist/cli.js", "mcp"]
    }
  }
}
```

`run_agent` roda em rascunho por padrao.

## A2A

O daemon publica o cartao em `/.well-known/agent.json`, com um skill por agente,
e aceita JSON-RPC em `/a2a` com `message/send` e `tasks/get`. A credencial e o
token do daemon em `Authorization: Bearer`.

## OpenTelemetry

Com `otel_endpoint`, cada chamada ao modelo e cada ferramenta vira um span OTLP
com atributos `gen_ai.*`.

## Standard Webhooks

Webhooks de saida em `webhooks.json` sao assinados no formato Standard Webhooks.

## Formatos do Claude Code

O daemon le skills (`SKILL.md`), plugins, agentes de plugin, hooks e servidores
MCP no formato do Claude Code, e importa os servidores MCP que voce ja
configurou la.

## Midia

Imagens anexadas sao guardadas uma vez, por hash, no SQLite. As mensagens levam
so a referencia, e o daemon serve o arquivo em `/media/<hash>` para a interface
local.
