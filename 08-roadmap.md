# 08. Roadmap

Fases curtas, cada uma com algo usavel no fim. Estimativas sao para uma
pessoa em tempo parcial.

## Fase 0. Planejamento

Estado: concluida em 2026-09-09.

Entregas:

- Esta documentacao.
- ADRs 0001 a 0004.
- Repositorios criados com README e estrutura de pastas.

Aceite: o dono da iniciativa leu e aprovou os documentos 01 a 08.

## Fase 1. Core e daemon por CLI

Estado: codigo entregue em 2026-09-09 com `core` (33 testes), `agents`
(tres perfis, precos, politicas, roteamento, uma skill) e `daemon` (CLI e
servidor WebSocket), incluindo compactacao, redacao de segredos, roteamento
por regra, ativacao de skills, MCP por stdio e HTTP e escalada para
`fallback_agent`. Pendente: OAuth, recursos e prompts MCP, e a validacao com
chaves reais dos aceites 1 a 4, que depende de preencher os precos do
DeepSeek e de ter o Ollama alcancavel pelo daemon.

Duracao alvo: 3 semanas.

Entregas:

- `core`: `ProviderAdapter` com `anthropic` e `openai-compatible`;
  `AgentRunner`; `Ledger` e `Budget` sobre SQLite; `Pricing`;
  `ToolRegistry` com ferramentas nativas; `McpBridge`; carregador de perfis.
- `agents`: os tres perfis, `pricing.json`, `policies/`, `mcp.json` de
  exemplo.
- `daemon`: servidor WebSocket local, sessoes, runs, aprovacoes por CLI,
  broadcast de eventos, `cost report`.
- MCP completo: stdio e HTTP streamable, tools, resources e prompts,
  OAuth 2.1 para servidores remotos.
- Agent Skills: carregamento sob demanda por `/nome` e `load_skill`.
- Funil de validacao e reparo de chamadas de ferramenta, com escalada
  para `fallback_agent`.
- Roteamento por regra em `agents/routing.json`.
- CLI de desenvolvimento: `agent-hub-daemon chat --agent deepseek-dev
  --workspace /caminho`.

Aceite:

1. Cada um dos tres agentes conclui uma tarefa real de edicao de arquivo
   com aprovacao pelo CLI.
2. O ledger mostra tokens e custo por chamada, com cache lido corretamente
   na Anthropic e no DeepSeek.
3. Um orcamento de run de 0.05 USD interrompe o `claude-arquiteto` no meio e
   preserva a resposta parcial.
4. Uma ferramenta MCP de leitura funciona com os tres agentes.
5. Testes de mapeamento de uso passam com fixtures reais de cada provedor.
6. Uma skill escrita para o Claude Code carrega e e usada pelos tres
   agentes sem alteracao no arquivo.
7. O `local-leitor` com uma chamada de ferramenta malformada passa pelo
   reparo e, esgotado, escala para o `deepseek-dev` com o custo registrado.

## Fase 2. Interface web e desktop

Estado: `web` entregue em 2026-09-09 (sessoes, chat com streaming,
aprovacoes, custos, agentes, automacoes, conexao direta ou pelo relay com
reconexao e catch-up). Agendamentos entregues no daemon. `desktop` escrito
como esqueleto Tauri 2 e nao compilado por falta de Rust na maquina de
desenvolvimento. Pendente: compilar o desktop, empacotar o daemon como
sidecar e validar os aceites 1 a 4 em instalacao real.

Duracao alvo: 3 semanas.

Entregas:

- `web`: chat com streaming, lista de sessoes, aprovacoes inline, painel de
  custo, cadastro de orcamento. PWA instalavel.
- `desktop`: Tauri com sidecar do daemon, bandeja, instaladores para os tres
  sistemas.
- Agendamentos por cron com fuso IANA, modo rascunho, tela "Automacoes" e
  interruptor geral.

Aceite:

1. Instalar no Windows e no Linux e usar os tres agentes sem terminal.
2. Abrir o mesmo daemon no navegador em `127.0.0.1` e ver a mesma sessao
   atualizando ao vivo nas duas janelas.
3. Painel de custo bate com `cost report` do CLI.
4. Um agendamento diario roda em modo rascunho, respeita o orcamento e
   aparece como sessao com origem `schedule`.

## Fase 3. Acesso remoto e mobile

Estado: `relay` entregue em 2026-09-09 com multiplexacao por canal, limite
de taxa e entrada de webhooks. Daemon liga ao relay com reconexao. Gatilhos
`gitlab`, `github` e `generic` (Standard Webhooks) com filtro, dedupe e
orcamento, verificados de ponta a ponta. Webhooks de saida em Standard
Webhooks. Canal Telegram funcional. Pendente: emparelhamento por QR code,
notificacao push do PWA, deploy do relay com TLS e validacao dos aceites em
rede real.

Duracao alvo: 2 semanas.

Entregas:

- `relay` em producao com TLS.
- Emparelhamento por QR code.
- PWA no celular com seletor de dispositivo, aprovacoes e notificacao push.
- Catch-up por `seq` e leitura offline.
- Gatilhos externos pelo relay: fontes `gitlab`, `github`, `jira` e
  `generic` no formato Standard Webhooks, com filtro, dedupe e orcamento.
- Webhooks de saida em Standard Webhooks.
- Canal Telegram no repositorio `channels`.

Aceite:

1. Do celular, fora da rede de casa, criar sessao no PC, pedir uma alteracao
   de codigo, aprovar o comando e ver o diff aplicado.
2. Fechar o app no celular no meio do run, abrir o desktop e ver o run
   concluido.
3. Token revogado derruba o celular na hora.
4. Abrir um MR no GitLab dispara o `claude-arquiteto` em modo rascunho e o
   resultado chega como sessao e como mensagem no Telegram.
5. Entrega repetida do mesmo webhook nao cria segunda sessao.

## Fase 4. Eficiencia

Duracao alvo: continua.

Estado em 2026-09-09: `core` com 49 testes. Tudo que nao depende de
ambiente externo foi entregue.

1. Compactacao propria com resumo pelo agente local e poda de resultados de
   ferramenta. Entregue.
2. Delegacao entre perfis com custo agregado no painel. Entregue.
3. Relatorios: cache por agente, sessoes clicaveis ordenadas por custo e
   exportacao CSV. Entregue.
4. Sandbox por container: perfil com `sandbox.image` roda `run_command` em
   `docker run` com o workspace montado e sem rede por padrao; execucao
   `allow` sem sandbox cai para `ask`. Entregue.
5. Cifra de ponta a ponta no relay com AES-GCM derivada do token de conta.
   Entregue e verificada.
6. Classificador de intencao pelo agente local como fallback das regras.
   Entregue.
7. Avaliar compactacao do lado do servidor da Anthropic. Pendente, depende
   de chave real para medir.
8. Plugins no layout do Claude Code e hooks com adaptador, por caminho local
   ou por git (`agent-hub-daemon plugins sync`). Entregue.
9. Fases por perfil e workflows declarativos com custo maximo calculavel,
   schema de saida e retry limitado. Entregue.
10. Servidor MCP exposto pelo daemon e exportador OpenTelemetry por
    OTLP/HTTP. Entregues. A2A nas duas direcoes pendente.
11. Canal Slack. Retirado do escopo pelo dono da iniciativa.
12. Emparelhamento por link e QR, notificacao push do PWA, recursos e
    prompts MCP. Entregues. OAuth em MCP remoto pendente, depende de um
    servidor com OAuth para testar.

Desktop: compilado para Linux em Docker (`desktop/scripts/build-linux-docker.sh`),
gerando deb, rpm e AppImage. Windows e macOS exigem Rust nessas plataformas.

## Proximo passo recomendado

Validar com chaves reais. Os precos ja estao preenchidos; falta exportar
`ANTHROPIC_API_KEY`, `DEEPSEEK_API_KEY` e `OPENAI_API_KEY` no ambiente do
daemon, deixar o Ollama alcancavel, e rodar os aceites 1 a 4 da fase 1 pelo
CLI e pela interface (`./dev.sh` na raiz sobe tudo em localhost). Depois:
A2A, OAuth em MCP, compactacao do servidor da Anthropic, e o desktop para
Windows e macOS.

## Riscos conhecidos

| Risco | Mitigacao |
|-------|-----------|
| Tool calling fraco em modelo local | Poucas ferramentas no perfil, validacao de JSON, `deny` em escrita |
| Precos mudam e o ledger fica errado | `pricing_version` por linha e recalculo sob demanda |
| Relay vira ponto unico de falha | URL manual de daemon para Tailscale, relay sem estado facil de subir de novo |
| Sidecar do daemon nao sobe em alguma plataforma | Daemon tambem instala como servico independente; desktop apenas conecta |
| Cache da Anthropic invalidado sem perceber | Alerta no painel quando `cache_read` fica zero em chamadas repetidas |
| Escopo cresce | Fase 4 e a unica aberta. Tudo novo entra nela, nunca nas anteriores |
