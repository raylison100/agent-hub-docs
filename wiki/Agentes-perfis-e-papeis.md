# Agentes, perfis e papeis

Tudo fica em texto na pasta de agentes (`agents_dir`). Mudou um arquivo, use
**Recarregar configuracao** em Configuracoes, Conexao, ou `make reiniciar`.

## Perfil: um modelo com suas regras

Um arquivo Markdown por agente em `profiles/`. O frontmatter configura, o corpo
e o prompt de sistema.

```markdown
---
name: deepseek
description: DeepSeek V4 Flash. Implementacao do dia a dia
provider: deepseek
model: deepseek-v4-flash
reasoning: medium
max_output: 16000
reasoning_budget: 16000
max_steps: 30
tools:
  native: [list_dir, read_file, search, edit_file, write_file, run_command, git]
  mcp: []
routing:
  capabilities:
    "*": 0.7
    implementar: 0.8
policy: padrao
budget:
  run_usd: 0.30
  session_usd: 3.00
context:
  window: 1000000
  compact_at: 0.5
  summarizer: qwen3
delegates: [qwen3]
fallback_agent: claude
---

Voce implementa mudancas pequenas e objetivas...
```

| Campo | Para que serve |
|---|---|
| `provider`, `model` | `anthropic`, `deepseek`, `gemini`, `openai` ou `ollama`, e o modelo da tabela de precos |
| `reasoning`, `reasoning_budget` | esforco de raciocinio e quanto dele soma no teto de saida |
| `max_output`, `max_steps` | teto de resposta e de passos do laco |
| `tools.native`, `tools.mcp` | ferramentas nativas e conectores que o agente pode usar |
| `routing.capabilities` | nota de 0 a 1 por intencao; `*` vale para as outras |
| `routing.latency` | `interativo`, `lote` ou `ambos` |
| `routing.vision` | aceita imagem |
| `policy` | politica de risco em `policies/` |
| `budget` | teto por run e por sessao |
| `context` | janela, quando compactar e quem resume |
| `delegates` | agentes para quem pode delegar subtarefa |
| `fallback_agent` | para quem escala quando erra chamada de ferramenta demais |
| `phases` | fases com ferramentas restritas, ver o README do daemon |
| `sandbox` | `run_command` dentro de container |

Os perfis que acompanham o projeto: `claude`, `deepseek`, `gemini`, `openai` e
`qwen3` (local).

## Papel: o trabalho separado do modelo

Um papel diz **o que fazer** (prompt, ferramentas, politica, skills) e **em quais
modelos pode rodar**. O roteador so escolhe entre esses modelos.

```markdown
---
name: revisor
description: Revisa mudanca de codigo procurando defeito, risco e falta de teste
models: [deepseek, gemini, openai, claude]
tools:
  native: [list_dir, read_file, search, git]
  mcp: []
skills: [revisar-mr]
policy: padrao
---

Voce revisa mudanca de codigo. Voce nao altera arquivo nenhum...
```

Assim a mesma revisao pode sair num modelo de centavos ou num de fronteira, sem
duplicar o prompt.

## Skills

Pastas em `skills/` no formato Agent Skills (`SKILL.md`). O prompt lista as
disponiveis e o agente carrega com `load_skill` quando a tarefa pede; regras de
ativacao podem carregar sozinhas.

## Politicas

`policies/*.json` define `read`, `write` e `exec` como `allow`, `ask` ou `deny`.
Comando destrutivo pede aprovacao em qualquer politica. `budgets.json` define o
teto mensal global, o de automacao e o diario por agente.

## Plugins do Claude Code

`plugins.json` aponta para plugins no layout do Claude Code: o daemon le skills,
comandos, agentes, servidores MCP e hooks. `overrides.json` escolhe provedor e
modelo dos agentes que vem de plugin.

O jeito mais facil e a tela **Configuracoes, Plugins**:

1. Em **Instalados no Claude Code**, clique em **Adicionar** no plugin que voce
   ja usa no Claude Code. O daemon prefere a pasta de origem do marketplace local
   a copia em cache, entao um `git pull` na origem ja atualiza o plugin.
2. Ou cole a pasta do plugin ou a URL git (com branch ou tag opcional) em
   **Adicionar por pasta ou git**. Repositorio git e clonado em
   `agents/.plugins`.
3. Se o plugin pede configuracao (uma chave de API, por exemplo), a tela mostra
   qual variavel falta e leva para **Chaves**.
4. **Criar papel com as skills** grava um papel em `roles/` com as skills, os
   servidores MCP e as ferramentas de escrita do plugin, para escolher na sessao.

Na mesma tela da para desligar um plugin sem tira-lo da lista e remove-lo (a
pasta nao e apagada). O arquivo continua editavel a mao:

```json
{ "plugins": [{ "path": "/caminho/do/marketplace/plugins/meu-plugin" }] }
```

- As skills entram com o prefixo do plugin (`meu-plugin:nome-da-skill`) e ficam
  disponiveis para o perfil ou papel que as listar em `skills`.
- Ao carregar uma skill, o agente recebe a pasta dela e pode ler seus arquivos
  (modelos, referencias, scripts) pelo caminho absoluto. Escrever continua so no
  workspace. `${CLAUDE_SKILL_DIR}` e `${CLAUDE_PLUGIN_ROOT}` no texto da skill
  viram os caminhos reais.
- Os servidores MCP do plugin se chamam `meu-plugin-servidor`. Campos de
  configuracao do usuario do plugin (`${user_config.gemini_api_key}`) sao lidos
  da chave de mesmo nome em maiusculas cadastrada em Chaves (`GEMINI_API_KEY`).
- Um jeito pratico de usar: um papel em `roles/` com as skills, o servidor MCP e
  as ferramentas de escrita, escolhido na hora de abrir a sessao.

## Precos

`pricing.json` tem o preco por milhao de tokens de cada modelo, com cache e
desconto por horario. Modelo fora da tabela bloqueia o run de proposito. Um
agendamento mensal lembra de conferir a tabela.
