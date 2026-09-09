# 09. Extensoes: MCP, skills, plugins e hooks

Quatro formas de estender o produto sem mudar o codigo. Todas entram pelo
mesmo registro no daemon e obedecem a politica de aprovacao e ao ledger.

## MCP

Complementa o que esta em `07-agentes.md`.

### Transportes

| Transporte | Uso | Autenticacao |
|------------|-----|--------------|
| stdio | servidor local iniciado pelo daemon | variaveis de ambiente por referencia |
| HTTP streamable | servidor remoto ou conector de terceiros | bearer fixo ou OAuth 2.1 |

OAuth: o daemon abre o navegador no desktop para o consentimento e guarda o
token de atualizacao no gerenciador de segredos do sistema. No celular, o
cliente mostra o link e o daemon espera o callback pelo relay. Tokens nunca
saem do daemon.

### Primitivas suportadas

| Primitiva | Como aparece |
|-----------|--------------|
| tools | ferramentas no registro, com classificacao de risco |
| resources | anexaveis a uma mensagem pela interface, entram como conteudo, nunca automaticamente |
| prompts | atalhos `/servidor:prompt` na interface, viram mensagem do usuario |

Recursos e prompts nao entram no prompt de sistema. Isso mantem o cache
estavel e o custo previsivel.

### Ciclo de vida

- Um servidor sobe na primeira sessao que o usa e fica vivo ate ficar
  ocioso por `idle_timeout` (padrao 15 minutos).
- Falha ao subir marca o servidor como indisponivel na interface e o
  perfil que depende dele avisa antes de iniciar o run.
- Lista de ferramentas e cacheada por servidor e invalidada por
  `notifications/tools/list_changed`.

## Skills

Instrucoes sob demanda em Markdown, no mesmo formato do Claude Code:
pasta com `SKILL.md`, frontmatter com `name` e `description`, corpo com as
instrucoes, arquivos auxiliares ao lado.

```
agents/skills/
  revisar-mr/
    SKILL.md
    checklist.md
```

Regras:

- A lista de skills disponiveis entra no prompt de sistema apenas como
  nome e descricao, uma linha cada. O corpo so entra quando a skill e
  invocada.
- Invocacao pelo usuario com `/nome` ou pelo agente com a ferramenta nativa
  `load_skill`. Nos dois casos o corpo entra como mensagem do usuario, no
  fim do contexto, preservando o cache do prefixo.
- Um perfil escolhe quais skills enxerga em `skills: [...]`. Sem a chave,
  nenhuma.
- O ledger marca o run com as skills carregadas, para medir quanto cada
  uma custa em tokens.

Compatibilidade: skills escritas para o Claude Code funcionam quando usam
apenas instrucoes e arquivos. Skills que dependem de ferramentas
exclusivas do Claude Code ou de comportamento de um modelo especifico
podem degradar em outros provedores. O carregador avisa quando encontra
referencias a ferramentas que o perfil nao tem.

## Plugins

Um plugin e uma pasta que agrupa skills, perfis de agente, servidores MCP
e hooks. O layout segue o do Claude Code para reaproveitar os plugins que
o usuario ja tem:

```
plugin/
  .claude-plugin/plugin.json   nome, versao, descricao
  skills/                      uma pasta por skill
  agents/                      perfis em Markdown
  commands/                    tratados como skills invocaveis por /nome
  .mcp.json                    servidores MCP
  hooks/hooks.json             hooks
```

Instalacao: `agents/plugins.json` lista caminhos locais ou repositorios
git. O daemon clona ou le, registra tudo com prefixo `plugin:` e mostra na
interface o que cada plugin trouxe.

Regras de adaptacao:

- Perfil de agente vindo de plugin nao define provedor nem modelo. O
  usuario define em `agents/overrides/<plugin>.json` ou aceita o padrao do
  daemon. Isso evita que um plugin force um modelo caro.
- Servidores MCP de plugin obedecem a classificacao de risco padrao
  (`*: write`) ate o usuario classificar.
- Um plugin pode ser desligado inteiro pela interface sem remover.

## Hooks

Comandos que rodam em pontos do ciclo de vida e podem bloquear a acao.

| Evento | Quando | Pode bloquear |
|--------|--------|---------------|
| `run.start` | antes da primeira chamada ao modelo | sim |
| `tool.before` | antes de executar qualquer ferramenta | sim |
| `tool.after` | depois do resultado, antes de voltar ao modelo | pode alterar o resultado |
| `run.end` | ao terminar, com custo total | nao |
| `budget.exceeded` | ao estourar orcamento | nao |

Formato proprio: o hook recebe JSON na entrada padrao e responde com JSON
`{ "decision": "allow" | "deny", "reason": "...", "output": ... }`. Codigo
de saida diferente de zero e tratado como `deny`.

Compatibilidade com hooks do Claude Code: um adaptador traduz os eventos
`PreToolUse`, `PostToolUse` e `Stop` para os equivalentes acima e converte
o JSON de entrada e saida. Eventos sem equivalente sao ignorados com aviso
no carregamento.

Hooks sao a forma prevista para portoes de qualidade, como bloquear commit
com segredo ou exigir teste antes de `git push`.

## Onde cada coisa vive

| Tipo | Local | Escopo |
|------|-------|--------|
| MCP global | `agents/mcp.json` | todos os perfis que listarem |
| MCP de plugin | `plugin/.mcp.json` | idem, com prefixo |
| Skills | `agents/skills/`, `plugin/skills/` | perfis que listarem |
| Hooks | `agents/hooks.json`, `plugin/hooks/hooks.json` | global ou por perfil via `hooks: [...]` |
| Plugins | `agents/plugins.json` | daemon |

## Custo

Tudo que entra no prompt de sistema e listado uma vez, estavel e curto.
Corpo de skill, recurso MCP e resultado de hook entram no fim da conversa,
sob demanda. O painel de custo mostra tokens por skill e por servidor MCP
para o usuario decidir o que manter.
