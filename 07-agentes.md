# 07. Agentes

## Formato do perfil

Um arquivo Markdown por agente em `agents/profiles/`. Frontmatter YAML com a
configuracao, corpo com o prompt de sistema. O formato e compativel com
`.claude/agents` do Claude Code para os campos comuns, para que o mesmo
arquivo sirva nos dois lugares.

```markdown
---
name: nome-do-agente
description: uma linha, usada na lista e por um futuro roteador
provider: anthropic | deepseek | openai | ollama
model: identificador no provedor
reasoning: low | medium | high | max
max_output: 16000
max_steps: 30
tools:
  native: [read_file, search, edit_file, run_command]
  mcp: [gitlab, jira]
policy: nome-em-agents/policies
budget:
  run_usd: 0.50
  session_usd: 5.00
context:
  window: 200000
  compact_at: 0.7
  summarizer: local-leitor
delegates: [local-leitor]
cache:
  system_ttl: 1h
provider_options:
  keep_alive: 30m
  num_ctx: 32768
---

Prompt de sistema aqui. Estavel, sem data, sem id.
```

Campos por provedor vao em `provider_options` e sao repassados pelo
adaptador sem interpretacao. Tudo fora disso e neutro.

## Os tres agentes iniciais

### claude-arquiteto

Tarefas de raciocinio pesado: desenho de solucao, revisao de MR, refatoracao
que cruza modulos, decisoes de arquitetura.

```yaml
name: claude-arquiteto
provider: anthropic
model: claude-opus-5
reasoning: high
max_output: 32000
max_steps: 40
tools:
  native: [list_dir, read_file, search, edit_file, write_file, run_command, git]
  mcp: []
policy: padrao
budget:
  run_usd: 2.00
  session_usd: 15.00
context:
  window: 1000000
  compact_at: 0.5
  summarizer: local-leitor
delegates: [local-leitor]
cache:
  system_ttl: 1h
```

Notas de custo: e o agente mais caro. `delegates` manda leitura e busca para
o agente local. O prompt de sistema e a lista de ferramentas ficam em cache
com TTL de uma hora, porque sessoes de arquitetura duram mais que cinco
minutos entre mensagens.

### deepseek-dev

Trabalho de implementacao do dia a dia: escrever funcao, corrigir teste,
ajustar endpoint. Bom custo por token e tool calling adequado.

```yaml
name: deepseek-dev
provider: deepseek
model: deepseek-chat
reasoning: medium
max_output: 8000
max_steps: 30
tools:
  native: [list_dir, read_file, search, edit_file, write_file, run_command, git]
  mcp: []
policy: padrao
budget:
  run_usd: 0.30
  session_usd: 3.00
context:
  window: 128000
  compact_at: 0.7
  summarizer: local-leitor
provider_options:
  base_url: https://api.deepseek.com
```

Notas de custo: `reasoning: high` troca para `deepseek-reasoner`, que custa
mais e pensa mais. Fica em `medium` por padrao e o usuario sobe por sessao
quando a tarefa pedir.

### local-leitor

Modelo local via Ollama. Leitura, busca, resumo, classificacao e servico de
compactacao para os outros dois. Custo zero por token.

```yaml
name: local-leitor
provider: ollama
model: qwen3:14b
reasoning: low
max_output: 4000
max_steps: 15
tools:
  native: [list_dir, read_file, search]
  mcp: []
policy: somente-leitura
budget:
  run_usd: 0
context:
  window: 32768
  compact_at: 0.8
provider_options:
  base_url: http://127.0.0.1:11434/v1
  keep_alive: 30m
  num_ctx: 32768
```

Notas: poucas ferramentas de proposito. Modelos locais erram com listas
grandes. O modelo e um ponto de partida; o usuario troca conforme a
maquina. Um modelo que nao suporta tool calling e recusado no carregamento
com mensagem clara.

## Politicas iniciais

`agents/policies/padrao.json`

```json
{ "read": "allow", "write": "ask", "exec": "ask" }
```

`agents/policies/somente-leitura.json`

```json
{ "read": "allow", "write": "deny", "exec": "deny" }
```

## MCP

`agents/mcp.json` lista servidores com comando, argumentos, variaveis de
ambiente por referencia (nunca o valor) e classificacao de risco por
ferramenta ou por prefixo de nome:

```json
{
  "servers": {
    "gitlab": {
      "command": "node",
      "args": ["/caminho/mcp-gitlab/dist/index.js"],
      "env": { "GITLAB_TOKEN": "$GITLAB_TOKEN" },
      "risk": { "gitlab_get_*": "read", "gitlab_list_*": "read", "*": "write" }
    }
  }
}
```

O daemon sobe um servidor MCP uma vez e o compartilha entre sessoes. Um
perfil so ve os servidores listados em `tools.mcp`. Ferramentas de um
servidor MCP entram no prompt apenas para aquele agente, o que mantem a
lista de ferramentas curta e o cache estavel.

## Validacao no carregamento

O daemon valida cada perfil com um schema Zod e recusa o arquivo inteiro se
algum campo estiver errado. Erros aparecem na interface, na lista de
agentes, com o caminho do arquivo e a linha.
