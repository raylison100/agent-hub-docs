# 10. Automacao: agendamentos, gatilhos externos e canais

Tres formas de um run comecar sem alguem digitar na interface. Todas rodam
no daemon, com orcamento obrigatorio e politica de aprovacao mais restrita
que a interativa, porque nao ha ninguem olhando.

## Regras comuns a toda automacao

- **Orcamento obrigatorio.** Toda automacao declara `budget.run_usd` e
  `budget.day_usd`. Sem isso o daemon recusa cadastrar. Existe ainda um
  teto global `automation.month_usd` em `agents/policies/budgets.json`.
- **Sem `ask` pendurado.** Em run automatico, uma ferramenta com decisao
  `ask` nao espera para sempre. O daemon envia `approval.required` aos
  clientes com prazo, e no vencimento aplica `on_timeout`, que por padrao e
  `deny`. O agente recebe o erro e decide como terminar.
- **Modo rascunho.** Automacao pode declarar `mode: draft`. Nesse modo
  escrita e execucao sao `deny` e o agente termina com um plano ou um diff
  proposto, que a pessoa aplica depois pela interface com um clique.
- **Uma execucao por vez.** Uma automacao nunca roda em paralelo consigo
  mesma. Disparo enquanto roda entra em fila ou e descartado, conforme
  `overlap: queue | skip`.
- **Interruptor geral.** `automation.enabled` no config do daemon e um
  botao na interface pausam tudo. Runs em andamento terminam, novos nao
  comecam.
- **Origem registrada.** Sessoes criadas por automacao carregam `origin`
  (`schedule`, `trigger`, `channel`) e o id da automacao. O ledger e os
  relatorios filtram por isso.

## Agendamentos

Runs periodicos com expressao cron, executados pelo daemon.

```json
{
  "id": "revisao-mrs-manha",
  "cron": "0 8 * * 1-5",
  "timezone": "America/Sao_Paulo",
  "agent": "deepseek-dev",
  "role": "revisor",
  "workspace": "/home/usuario/Projects/meu-projeto/api",
  "prompt": "Liste os MRs abertos atribuidos a mim e resuma o que cada um precisa para ser aprovado.",
  "mode": "draft",
  "budget": { "run_usd": 0.20, "day_usd": 0.50 },
  "overlap": "skip",
  "missed": "skip",
  "notify": ["telegram"]
}
```

- `role` (opcional): papel de `agents/roles` usado no run e gravado na
  sessao, com os modelos, ferramentas, skills e politica dele. Sem `role`, o
  papel sai das regras do `routing.json` como num pedido comum.
- `notify` (opcional): canais que recebem a resposta final do agente
  (`telegram`; `slack`, `discord` e `whatsapp` quando existirem). Os canais
  sao configurados em Configuracoes > Canais e rodam dentro do daemon. O texto
  vai para a primeira pessoa permitida que ja falou com o bot, e a conversa
  fica presa nessa sessao: a resposta de quem recebeu continua a conversa com o
  agente (ex.: aprovar itens propostos). `/nova` solta a conversa. Resposta que
  comeca com `[sem-aviso]` nao vai para o canal (rotina que nao achou trabalho).
  Linhas de imagem em markdown `![legenda](caminho)` com arquivo dentro do
  workspace sao enviadas como foto nos canais que aceitam imagem.
- `mode`: `draft` bloqueia escrita e comandos. `normal` usa a politica do
  papel; como ninguem esta olhando, o que for `ask` espera aprovacao ate o
  prazo e depois e negado.
- `missed`: o que fazer com disparos perdidos porque a maquina estava
  desligada. `skip` ignora, `run_once` executa uma vez ao ligar.
- Agendamentos ficam na tabela `schedules` do SQLite e podem ser criados
  pela interface ou por arquivo em `agents/schedules/*.json`. Arquivo e
  interface sao mesclados; conflito de id prefere o arquivo.
- O resultado vira uma sessao normal, visivel em todos os clientes, com
  notificacao push no celular quando habilitada.

Agendamento de uso unico (`at` em vez de `cron`) cobre "roda isso hoje as
22h".

## Gatilhos externos

Terceiros iniciam um run: GitLab ao abrir MR, GitHub ao falhar um
workflow, Jira ao mover um card, qualquer sistema com webhook.

O daemon nao tem porta publica. O relay recebe o webhook e encaminha:

```
POST https://relay.exemplo.com/hooks/<device_id>/<trigger_id>
```

Validacao no relay: assinatura HMAC com segredo do gatilho, limite de taxa
por origem, tamanho maximo do corpo. O relay encaminha ao daemon como
quadro `trigger.fire` com o corpo bruto. O daemon revalida a assinatura,
porque o relay nao e confiavel por desenho.

Sem relay, com Tailscale, o daemon pode ligar um receptor de webhook
apenas na interface da VPN.

Configuracao:

```json
{
  "id": "mr-aberto",
  "source": "gitlab",
  "secret_ref": "$TRIGGER_MR_SECRET",
  "filter": { "object_kind": "merge_request", "object_attributes.action": "open", "project.path_with_namespace": "meu-grupo/api" },
  "agent": "claude-arquiteto",
  "workspace": "/home/usuario/Projects/meu-projeto/api",
  "prompt": "Revise o MR {{object_attributes.iid}} ({{object_attributes.title}}). Leia o diff, aponte defeitos e publique o resultado como comentario no MR.",
  "mode": "draft",
  "budget": { "run_usd": 1.00, "day_usd": 5.00 },
  "dedupe": "object_attributes.last_commit.id",
  "overlap": "queue"
}
```

- `filter` compara campos do corpo por caminho. Corpo que nao casa e
  descartado sem criar sessao nem gastar token.
- `prompt` aceita campos do corpo entre chaves duplas. Conteudo de
  terceiros entra como dado na mensagem do usuario, delimitado, nunca no
  prompt de sistema. O agente e instruido a tratar esse conteudo como
  informacao, nao como ordem.
- `dedupe` evita rodar duas vezes para a mesma entrega. Ids processados
  ficam na tabela `trigger_deliveries` por 30 dias.
- Fontes com formato conhecido (`gitlab`, `github`, `jira`, `generic`) tem
  validador de assinatura pronto. `generic` usa HMAC SHA-256 do corpo.

Um gatilho tambem pode vir de dentro: `tool.after` de um hook, ou o fim de
outro run, encadeando automacoes. Fica para depois da primeira versao.

## Acoes em terceiros

O caminho principal ja existe: ferramentas MCP (GitLab, GitHub, Jira,
banco). Duas adicoes:

- Ferramenta nativa `http_request`, restrita a uma lista de hosts em
  `agents/policies/http.json`, com classificacao de risco por metodo (`GET`
  leitura, demais escrita). Cobre APIs sem servidor MCP.
- **Webhooks de saida** em `agents/webhooks.json`: `run.end`,
  `budget.exceeded` e `approval.required` podem chamar uma URL com o
  resumo. E assim que um agendamento avisa no Slack ou Telegram sem o
  daemon conhecer essas plataformas.

## Canais

Um canal e um cliente do daemon que vive em uma plataforma de mensagem:
Telegram, Slack, WhatsApp. O usuario manda uma mensagem, o canal cria ou
continua uma sessao, o agente responde na mesma conversa, e aprovacoes
viram botoes.

- O canal fala o mesmo protocolo WebSocket que a interface. Nao ha caminho
  especial.
- Cada canal e um processo separado, no repositorio `channels`, um por
  plataforma. Roda junto com o daemon ou em outra maquina via relay.
- Mensagens de canal sao do usuario, com a mesma politica interativa,
  porque ha alguem do outro lado. A diferenca e que a identidade do
  remetente e verificada contra uma lista de ids permitidos por canal.
  Qualquer outro remetente e ignorado.

Telegram entra primeiro por ser o mais simples de operar sem servidor
publico. Slack depois.

## Persistencia

Tabelas novas no SQLite do daemon:

- `schedules` (id, cron, at, timezone, agent, workspace, prompt, mode,
  budget_json, overlap, missed, notify_json, enabled, last_run_at,
  next_run_at)
- `triggers` (id, source, secret_ref, filter_json, agent, workspace,
  prompt, mode, budget_json, dedupe, overlap, enabled)
- `trigger_deliveries` (trigger_id, delivery_key, received_at, session_id)
- `automation_runs` (id, kind, automation_id, session_id, started_at,
  finished_at, status, cost_usd)

## Interface

Uma tela "Automacoes" com as tres listas, estado (ativa, pausada, rodando,
erro), ultimo run, custo do dia e do mes, e o interruptor geral. Cada item
abre o historico de runs com custo por execucao.

## Protocolo

Quadros novos, detalhados em `02-arquitetura.md`:

- Cliente para daemon: `schedule.list`, `schedule.upsert`,
  `schedule.delete`, `schedule.run_now`, `trigger.list`,
  `trigger.upsert`, `trigger.delete`, `automation.pause`,
  `automation.resume`.
- Relay para daemon: `trigger.fire`.
- Daemon para clientes: `automation.started`, `automation.finished`,
  `automation.error`.
