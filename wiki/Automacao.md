# Automacao

Trabalho que roda sem voce na frente: por horario, por evento de terceiro ou em
etapas fixas. Toda automacao tem orcamento proprio e comeca em modo rascunho.

## Agendamentos

Arquivos em `agents/schedules/*.json` ou cadastro pela tela Automacoes.

```json
{
  "id": "resumo-diario",
  "cron": "0 8 * * 1-5",
  "timezone": "America/Sao_Paulo",
  "agent": "qwen3",
  "workspace": "/home/usuario/Projects/meu-projeto",
  "prompt": "Resuma o que mudou no git log de ontem.",
  "mode": "draft",
  "budget": { "run_usd": 0.2, "day_usd": 0.5 },
  "overlap": "skip",
  "missed": "skip",
  "enabled": true
}
```

- `mode: draft` nega escrita e execucao; `normal` segue a politica do perfil.
- `overlap` decide o que fazer se o anterior ainda esta rodando.
- `missed` decide se roda o que perdeu enquanto a maquina estava desligada.
- O interruptor geral pausa todas as automacoes e sobrevive a reinicio.
- Como ninguem esta esperando, o roteador pesa mais o custo.

Vem com dois: revisao mensal da tabela de precos e auditoria mensal da memoria do
projeto, ambos no modelo local.

## Gatilhos

Um webhook de terceiro (GitLab, GitHub ou generico) dispara um run.

```json
{
  "id": "mr-aberto",
  "source": "gitlab",
  "secret_ref": "$TRIGGER_MR_SECRET",
  "filter": { "object_kind": "merge_request", "object_attributes.action": "open" },
  "agent": "claude",
  "workspace": "/home/usuario/Projects/meu-projeto/api",
  "prompt": "Revise o MR {{object_attributes.iid}} ({{object_attributes.title}}).",
  "mode": "draft",
  "budget": { "run_usd": 1.0, "day_usd": 5.0 },
  "dedupe": "object_attributes.last_commit.id",
  "overlap": "queue"
}
```

- A assinatura e conferida com o segredo de `secret_ref`, do jeito de cada fonte.
- `filter` descarta o que nao interessa sem criar sessao nem gastar token.
- `dedupe` evita rodar duas vezes para a mesma entrega.
- O conteudo do terceiro entra como dado delimitado na mensagem, nunca como
  instrucao do sistema.
- Chegam pelo relay em `POST /hooks/<dispositivo>/<gatilho>`, sem abrir porta.

## Workflows

Sequencias fixas em `agents/workflows/*.yaml`. O codigo decide a ordem; o modelo
so trabalha dentro de cada etapa.

- **tool**: roda uma ferramenta, sem modelo.
- **agent**: chama um agente com ferramentas restritas, limite de passos e,
  opcionalmente, `output_schema` validado.
- **gate**: para e pergunta a voce quando a confianca fica abaixo do limiar.
- **retry**: o unico laco permitido.

O custo maximo e calculado antes de rodar. Workflow interrompido continua do
ultimo checkpoint.

Vem com dois:

- `corrigir-teste`: roda os testes, pede a correcao e roda de novo.
- `deliberar`: especialista, critico e advogado do diabo em modelos diferentes
  analisam uma proposta, e o sintetizador decide; com confianca abaixo de 0,7, a
  decisao volta para voce.

```bash
agent-hub-daemon workflows deliberar --workspace /caminho --input proposta="migrar para X"
```

## Hooks

`hooks.json` roda comandos em eventos do ciclo de vida (antes e depois de
ferramenta, fim de run). O hook recebe JSON e responde `allow` ou `deny`. Ha um
catalogo pronto na interface, com formatar depois de escrever, conferir lint e
bloquear comando perigoso. Hooks de plugin do Claude Code sao traduzidos.

## Webhooks de saida

`webhooks.json` avisa sistemas externos de eventos do daemon, assinados no
formato Standard Webhooks.

## Subagentes

Um agente com `delegates` pode passar subtarefas para outros: esperando a
resposta (`delegate`) ou em segundo plano (`spawn` e `collect`), opcionalmente
numa worktree git isolada, em branch propria. O custo do filho entra no run pai.
