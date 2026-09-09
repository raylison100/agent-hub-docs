# 03. Repositorios

Oito repositorios independentes dentro da pasta `agent-hub/`. Cada um tem
seu proprio git, CI e versionamento. Dependencias entre eles passam por
pacote npm publicado em registro privado ou por `file:` durante o
desenvolvimento local.

## Grafo de dependencia

```
agents  <--  daemon  <--  desktop
core    <--  daemon
core    <--  web (apenas tipos do protocolo)
core    <--  channels (apenas tipos do protocolo)
relay   (independente)
docs    (independente)
```

## core

| Item | Valor |
|------|-------|
| Tipo | Biblioteca TypeScript, publicada como `@agent-hub/core` |
| Runtime | Node 22 LTS |
| Dependencias | `@anthropic-ai/sdk`, `openai`, `@modelcontextprotocol/sdk`, `zod`, `better-sqlite3` (apenas para o ledger) |
| Testes | Vitest, com adaptadores falsos para o loop e fixtures de resposta real de cada provedor para o mapeamento de uso |

Exporta: `ProviderAdapter` e implementacoes, `AgentRunner`, `Ledger`,
`Budget`, `Pricing`, `ToolRegistry`, `McpBridge`, `loadProfiles`, tipos do
protocolo.

Nao contem: servidor HTTP, interface, chaves de API, decisoes de politica de
aprovacao (recebe a politica como funcao).

## daemon

| Item | Valor |
|------|-------|
| Tipo | Servico Node, binario unico via `pkg` ou `node --experimental-sea` para virar sidecar do Tauri |
| Dependencias | `@agent-hub/core`, `fastify`, `@fastify/websocket`, `better-sqlite3` |
| Configuracao | `~/.agent-hub/config.toml`, variaveis de ambiente para chaves |

Responsabilidades:

- Subir servidor WebSocket local (padrao `127.0.0.1:47311`).
- Conectar ao relay quando configurado.
- Carregar perfis do repositorio `agents` (caminho configuravel).
- Gerenciar sessoes, runs, aprovacoes e broadcast de eventos.
- Executar ferramentas nativas com as restricoes de `06-execucao-remota.md`.
- Manter servidores MCP vivos e compartilha-los entre sessoes.
- Expor CLI minimo: `agent-hub-daemon start`, `status`, `pair` (gera token de
  dispositivo), `cost report`.

## relay

| Item | Valor |
|------|-------|
| Tipo | Servico Node pequeno, sem banco, deploy em qualquer VPS ou Fly.io |
| Dependencias | `ws`, `fastify` |
| Estado | Apenas em memoria: mapa de daemons registrados e clientes conectados |

Responsabilidades:

- Aceitar registro de daemon (`device_id` + segredo).
- Aceitar clientes (`account_token`) e listar dispositivos online.
- Encaminhar quadros entre cliente e daemon sem interpreta-los.
- Derrubar conexoes sem atividade e limitar taxa por IP.

Alternativa documentada em `adr/0004`: usar Tailscale e conectar direto no
daemon, sem relay. O protocolo e o mesmo.

## web

| Item | Valor |
|------|-------|
| Tipo | Aplicacao Vue 3 + Vite + TypeScript, PWA instalavel |
| Dependencias | `vue`, `pinia`, `vue-router`, `vite-plugin-pwa`, tipos de `@agent-hub/core` |
| Estilo | CSS proprio com tokens de tema. Sem framework de componentes pesado |

Telas iniciais:

- Lista de sessoes e seletor de dispositivo.
- Chat com streaming, chamadas de ferramenta expandiveis e aprovacoes inline.
- Painel de custo: run atual, sessao, dia, por agente e modelo.
- Cadastro de orcamentos.
- Visualizador de perfis de agente (somente leitura na fase 2).

O mesmo build atende desktop (dentro do Tauri), navegador e celular. A
diferenca e apenas a URL do daemon: local no desktop, relay nos outros.

## desktop

| Item | Valor |
|------|-------|
| Tipo | Tauri 2, Rust minimo |
| Papel | Janela nativa, bandeja do sistema, inicio automatico, subir e monitorar o daemon como sidecar, abrir o build do `web` |
| Instaladores | MSI e NSIS (Windows), DMG (macOS), AppImage e deb (Linux) |

Nao contem logica de agente. Se o daemon ja estiver rodando (instalado como
servico), o desktop apenas conecta.

## agents

| Item | Valor |
|------|-------|
| Tipo | Repositorio de dados, sem codigo |
| Conteudo | `profiles/*.md`, `pricing.json`, `mcp.json`, `policies/*.json` |

O daemon le este repositorio de um caminho local. Sincronizar entre maquinas
e `git pull`. Formato em `07-agentes.md`. Tambem guarda `skills/`,
`workflows/`, `schedules/`, `routing.json`, `plugins.json`, `hooks.json` e
`webhooks.json`, descritos em `09-extensoes.md`, `10-automacao.md` e
`11-determinismo.md`.

## channels

| Item | Valor |
|------|-------|
| Tipo | Um processo Node por plataforma de mensagem, comecando por Telegram |
| Dependencias | tipos do protocolo de `@agent-hub/core`, SDK oficial da plataforma |
| Papel | Cliente do daemon que vive em um chat: mensagem vira sessao, aprovacao vira botao |

Fala o mesmo protocolo WebSocket da interface. Verifica o remetente contra
uma lista de ids permitidos e ignora o resto. Detalhes em `10-automacao.md`.

## docs

Este repositorio. Planejamento, ADRs e roadmap.

## Convencoes comuns

- TypeScript estrito em todos os repositorios com codigo.
- Sem comentarios dentro de funcoes. No maximo um docblock curto acima.
- Sem emojis em codigo, commits, PRs ou logs.
- Commits em portugues, sem assinatura de ferramenta.
- Branch `main` protegida, trabalho em `feat/*` e `fix/*`.
- Cada repositorio tem `README.md` com como rodar, testar e publicar.
