# 04. Controle de custos

Este e o requisito que mais molda o desenho. Nenhuma chamada a modelo
acontece fora do caminho ledger e orcamento.

## Ledger

Uma linha por chamada ao provedor, gravada antes de qualquer outra coisa
acontecer com a resposta.

| Campo | Origem |
|-------|--------|
| `ts` | relogio do daemon |
| `session_id`, `run_id`, `step` | runner |
| `agent`, `provider`, `model` | perfil |
| `input`, `output`, `cache_read`, `cache_write`, `reasoning` | campo de uso da resposta, normalizado pelo adaptador |
| `cost_usd` | calculado com `pricing.json` na hora da gravacao |
| `pricing_version` | versao da tabela usada, para recalcular depois |
| `latency_ms` | runner |
| `stop_reason` | adaptador |

Regras:

- O adaptador nunca estima uso. Se o provedor nao devolver o campo, o valor
  fica nulo e a linha e marcada `usage_missing`. A interface mostra isso.
- Tokens de raciocinio contam como saida para custo, salvo quando a tabela de
  precos diz o contrario.
- Nada e apagado. Relatorios sao agregacoes do ledger.

## Tabela de precos

`agents/pricing.json`, versionada. Precos por milhao de tokens em USD.

```json
{
  "version": "2026-09-09",
  "models": {
    "anthropic/claude-opus-5":  { "input": 5.0, "output": 25.0, "cache_read": 0.5, "cache_write": 6.25 },
    "anthropic/claude-sonnet-5": { "input": 2.0, "output": 10.0, "cache_read": 0.2, "cache_write": 2.5 },
    "anthropic/claude-haiku-4-5": { "input": 1.0, "output": 5.0, "cache_read": 0.1, "cache_write": 1.25 },
    "deepseek/deepseek-chat":   { "input": null, "output": null, "cache_read": null, "note": "preencher com a tabela vigente do provedor" },
    "deepseek/deepseek-reasoner": { "input": null, "output": null, "cache_read": null },
    "ollama/*":                 { "input": 0, "output": 0, "cache_read": 0, "cache_write": 0 }
  }
}
```

- Valores da Anthropic acima vem da tabela oficial em 2026-06. Cache de
  leitura custa 0.1x da entrada, escrita de cache 1.25x. Confirmar antes de
  fechar a fase 1.
- DeepSeek cobra diferente para acerto e erro de cache. O adaptador separa os
  dois e a tabela precisa dos dois precos.
- Modelo local custa zero por token. Opcionalmente o perfil pode declarar
  `cost_per_hour` para estimar energia e amortizacao de hardware. Fica como
  melhoria da fase 4.
- Padrao curinga `ollama/*` cobre qualquer modelo local sem cadastro.
- Modelo sem preco na tabela bloqueia o run com erro claro. Nunca assume zero.

## Orcamentos

Quatro escopos, todos opcionais, todos verificados a cada chamada:

| Escopo | Chave | Exemplo |
|--------|-------|---------|
| `run` | id do run | maximo 0.50 USD ou 200k tokens por run |
| `session` | id da sessao | maximo 5 USD por sessao |
| `agent` | nome do perfil | `claude-arquiteto` maximo 20 USD por dia |
| `global` | conta | 100 USD por mes |

Verificacao antes de cada chamada:

```
estimativa = ultimo_input_da_sessao + tokens(novas mensagens) + perfil.maxOutput
custo_estimado = pricing.cost(model, estimativa)
para cada escopo:
  se gasto + custo_estimado > limite: parar com budget_exceeded
  se gasto + custo_estimado > 0.8 * limite: emitir budget_warning uma vez
```

Verificacao depois de cada chamada com o uso real. Se estourou, o run para
ali, com a resposta parcial preservada.

A interface pode responder `budget.override` para subir o limite daquele
escopo naquele momento. O override e registrado no ledger.

Valores padrao vem do perfil (`budget.run`, `budget.session`) e de
`agents/policies/budgets.json` para agente e global.

## Estimativa de tokens

- Anthropic: `countTokens` do SDK quando a estimativa importa (primeiro passo
  de um run ou depois de compactacao). Nas demais chamadas, usa o input da
  chamada anterior somado ao tamanho aproximado do que foi acrescentado.
- Demais provedores: apenas o metodo incremental acima. Aproximacao de 4
  caracteres por token para o acrescimo, corrigida pelo uso real da chamada
  seguinte.
- Nunca usar tokenizer local de outra familia. O erro entre familias passa de
  30% e a correcao pelo uso real e mais barata.

## Cache por provedor

Regra geral, valida para todos: prompt de sistema e lista de ferramentas
sao estaveis e vem primeiro; conteudo volatil vem depois. Nada de data, hora
ou id aleatorio no prompt de sistema.

| Provedor | Mecanismo | O que o adaptador faz |
|----------|-----------|-----------------------|
| Anthropic | Explicito, `cache_control` | Marca o prompt de sistema e a ultima ferramenta com `{type: "ephemeral"}`. Marca a ultima mensagem do usuario com TTL de 1h em sessoes longas. Verifica `cache_read_input_tokens` e alerta se ficar zero em chamadas repetidas |
| DeepSeek | Automatico por prefixo | Nada a enviar. Le `prompt_cache_hit_tokens` para o ledger |
| OpenAI | Automatico por prefixo | Le `cached_tokens` |
| Ollama | Cache de KV por sessao | Envia `keep_alive` e `num_ctx` no perfil para nao recarregar o modelo entre passos |
| vLLM | Prefix caching no servidor | Documentar flag de inicializacao no README do `agents` |

O painel de custo mostra a taxa de acerto de cache por agente. Uma taxa baixa
em um agente Claude e um defeito, nao um detalhe.

## Contexto e compactacao

Cada perfil declara `context.window` (tamanho do modelo) e
`context.compactAt` (fracao, padrao 0.7). Quando o input estimado passa do
limite:

1. Poda: remove resultados de ferramenta antigos, mantendo um resumo de uma
   linha por chamada.
2. Compactacao: resume a conversa ate o ponto de corte usando o modelo mais
   barato disponivel no perfil (`context.summarizer`, padrao o agente local).
3. O historico completo continua no SQLite. Apenas o que vai ao provedor
   muda.

Para Anthropic, avaliar na fase 4 a compactacao do lado do servidor (beta).
Ate la, o mecanismo proprio vale para todos.

## Roteamento

O agente e escolhido pelo usuario. Um perfil pode declarar `delegates` para
passar subtarefas a outro perfil mais barato (leitura de arquivos, busca,
resumo). O runner registra a delegacao no ledger com `parent_run_id`, e o
painel mostra o custo agregado do run pai.

Na fase 4, um agente `roteador` com modelo local classifica a tarefa e sugere
o perfil. Sempre sugestao, nunca troca automatica.

## Parametros de raciocinio por provedor

O perfil declara `reasoning` de forma neutra e o adaptador traduz:

| Perfil | Anthropic | OpenAI | DeepSeek | Ollama |
|--------|-----------|--------|----------|--------|
| `low` | `output_config.effort: low` | `reasoning_effort: low` | `deepseek-chat` | sem alteracao |
| `medium` | `effort: medium` | `medium` | `deepseek-chat` | sem alteracao |
| `high` | `effort: high` | `high` | `deepseek-reasoner` | `think: true` quando o modelo suporta |
| `max` | `effort: xhigh` | `high` | `deepseek-reasoner` | `think: true` |

Anthropic: pensamento adaptativo e o padrao nos modelos atuais, o adaptador
nao envia `budget_tokens`.

## Relatorios

Consultas prontas sobre o ledger, expostas por `cost.report`:

- Hoje, semana, mes: total, por agente, por modelo, por sessao.
- Por run: linha do tempo de custo passo a passo.
- Taxa de cache por agente e modelo.
- Top 10 sessoes mais caras.

Exportacao CSV do ledger para analise externa.
