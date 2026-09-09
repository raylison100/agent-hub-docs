# 11. Determinismo: o harness decide, o modelo executa

Principio: toda decisao que pode ser tomada por regra e tomada pelo
harness, antes ou depois da chamada ao modelo, e nunca delegada ao modelo.
O modelo recebe o menor problema possivel e a menor lista de opcoes
possivel. Isso compensa modelos fracos em tool calling, reduz tokens e
torna o comportamento reproduzivel entre provedores.

## Divisao de responsabilidade

| Decisao | Quem decide | Como |
|---------|-------------|------|
| Qual agente atende a tarefa | harness, com sugestao opcional de modelo | regras de roteamento |
| Quais ferramentas o modelo ve neste passo | harness | filtro por fase, politica e perfil |
| Qual skill entra no contexto | harness | regras de ativacao no frontmatter |
| Se uma chamada de ferramenta e valida | harness | JSON Schema, normalizacao, reparo |
| Se uma chamada de ferramenta pode rodar | harness | politica de aprovacao |
| Quando compactar, quanto gastar, quando parar | harness | orcamento e limites do perfil |
| Em que ordem as etapas de um trabalho acontecem | harness, quando ha workflow | workflow declarativo |
| O conteudo de cada etapa | modelo | chamada ao provedor |

## Roteamento por regra

`agents/routing.json` avaliado antes de qualquer chamada. Primeira regra
que casa vence. Sem regra casando, usa o agente escolhido pelo usuario.

```json
[
  { "when": { "workspace": "**/meu-projeto/**", "intent": "revisar" }, "agent": "claude-arquiteto" },
  { "when": { "files_touched": ["*.test.ts", "*Test.php"] }, "agent": "deepseek-dev" },
  { "when": { "prompt_tokens_lt": 400, "intent": "explicar" }, "agent": "local-leitor" }
]
```

`intent` vem de um classificador barato: primeiro por palavras chave
deterministas, e so se nao houver casamento, pelo agente local com saida
estruturada. O resultado do classificador e registrado no ledger com a
regra aplicada.

## Exposicao de ferramentas por fase

Um perfil pode declarar fases. Em cada fase o modelo ve apenas as
ferramentas daquela fase. A troca de fase e decidida pelo harness por
evento, nao pelo modelo.

```yaml
phases:
  - name: entender
    tools: [list_dir, read_file, search, load_skill]
    until: { tool_called: "plan" }
  - name: executar
    tools: [read_file, edit_file, write_file, run_command]
    until: { tool_called: "done" }
  - name: verificar
    tools: [run_command, read_file]
    until: { max_steps: 5 }
```

`plan` e `done` sao ferramentas nativas sem efeito, que existem para o
modelo sinalizar. Modelos locais se beneficiam muito: em vez de quinze
ferramentas, veem quatro.

## Ativacao de skills por regra

Alem de `/nome` e `load_skill`, o frontmatter de uma skill pode declarar
quando o harness a carrega sozinho:

```yaml
name: revisar-mr
activate:
  intent: [revisar]
  files: ["*.php"]
  workspace: "**/meu-projeto/**"
```

A skill entra no fim do contexto quando a regra casa, com registro no
ledger. O modelo nao precisa lembrar que ela existe.

## Validacao e reparo de chamadas de ferramenta

Toda chamada passa por um funil antes de executar:

1. **Parse.** Argumentos sao lidos com `JSON.parse`. Falha vira erro
   devolvido ao modelo com a mensagem exata.
2. **Schema.** Validacao contra o JSON Schema da ferramenta. Campos
   desconhecidos sao removidos; campos obrigatorios ausentes viram erro.
3. **Normalizacao.** Caminhos relativos viram absolutos dentro do
   workspace, barras sao unificadas, `~` e expandido. O modelo nunca e
   penalizado por detalhe de formato.
4. **Reparo limitado.** Ate `repair_attempts` (padrao 2) o harness devolve
   o erro e pede de novo. Depois disso o run para com `tool_call_invalid`.
5. **Escalada.** Se um perfil declara `fallback_agent`, o harness reenvia o
   mesmo contexto ao agente de fallback apos esgotar os reparos. Modelo
   local que nao consegue chamar `edit_file` escala para o DeepSeek, com o
   custo registrado.

Para provedores com suporte, o harness liga `strict: true` no schema da
ferramenta e saida estruturada quando a ferramenta so existe para devolver
JSON. Isso e configuracao do adaptador, invisivel ao perfil.

## Workflows declarativos

Para trabalho repetitivo, um workflow fixa a sequencia e o modelo so
preenche cada etapa. Arquivo em `agents/workflows/*.yaml`:

```yaml
name: corrigir-teste-quebrado
inputs: [test_path]
steps:
  - id: rodar
    tool: run_command
    args: { command: "vendor/bin/phpunit {{test_path}}" }
  - id: diagnosticar
    agent: local-leitor
    prompt: "Explique por que o teste falhou. Saida do teste: {{rodar.output}}"
    output_schema: { cause: string, files: [string] }
  - id: corrigir
    agent: deepseek-dev
    prompt: "Corrija a causa: {{diagnosticar.cause}}. Arquivos: {{diagnosticar.files}}"
    tools: [read_file, edit_file]
  - id: validar
    tool: run_command
    args: { command: "vendor/bin/phpunit {{test_path}}" }
    retry: { step: corrigir, max: 2, when: "exit_code != 0" }
```

- Etapas `tool` rodam sem modelo. Etapas `agent` chamam um perfil com
  ferramentas restritas a lista da etapa.
- `output_schema` forca saida estruturada e a validacao e do harness.
- `retry` e a unica forma de laco. Sem laco livre, o custo maximo e
  calculavel antes de comecar e aparece na interface.
- Workflows sao invocaveis pelo usuario, por agendamento e por gatilho.

## Reprodutibilidade

- Ordem de ferramentas e de skills no prompt e alfabetica e fixa.
- Prompt de sistema nunca contem data, hora ou id. Contexto volatil vai na
  mensagem do usuario.
- Onde o provedor aceita, o adaptador envia `temperature: 0` e `seed`
  fixo por sessao, configuraveis no perfil. Anthropic nos modelos atuais
  nao aceita esses parametros e o adaptador os omite.
- Cada run guarda a versao do perfil, das skills e da tabela de precos
  usados, para reproduzir depois.

## O que o modelo continua decidindo

Conteudo de codigo, explicacao, qual arquivo abrir dentro da fase, quando
sinalizar `plan` e `done`. O harness nao tenta prever isso. A meta e
reduzir o espaco de escolha, nao substituir o modelo.

## Custo

Cada mecanismo acima reduz tokens: menos ferramentas no prompt, skills so
quando casam, workflows sem laco livre, roteamento para o modelo local
sempre que a regra permitir. O painel mostra, por run, quais regras
determinaram o caminho.
