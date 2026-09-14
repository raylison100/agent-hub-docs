# Contexto do projeto e memoria

O agente sabe do seu projeto por quatro caminhos, todos dentro do workspace e
todos com teto de tokens.

## Instrucoes do projeto

O primeiro arquivo que existir entre `.claude/CLAUDE.md`, `CLAUDE.md`,
`AGENTS.md` e `.agent-hub/INSTRUCOES.md` entra sempre no prompt de sistema. Se
voce ja usa Claude Code, as mesmas instrucoes valem aqui.

## Glossario

`.agent-hub/glossario.md` entra sempre, para os agentes usarem as palavras do
seu time e nao sinonimos.

## Memoria

`.agent-hub/memory/` guarda um fato por arquivo, com cabecalho:

```markdown
---
name: porta-do-daemon
description: onde o daemon escuta
activate: {"keywords": ["porta", "daemon"]}
---

O daemon escuta em 127.0.0.1:47311.
```

- Item **com** `activate` entra so quando o pedido casa com as palavras ou os
  arquivos da regra.
- Item **sem** `activate` entra em toda conversa, e por isso custa sempre.
- A memoria vai dentro da mensagem que a ativou e nao se repete enquanto estiver
  no historico. Se o item mudar, a versao nova e entregue de novo.
- Os agentes escrevem com `memory_write`, sempre com run, agente e data.
- Um agendamento mensal audita a memoria contra o codigo.

## Base de conhecimento

`.agent-hub/knowledge/` recebe material do projeto: processos, notas,
transcricoes em `.md`, `.txt`, `.csv`, `.json` ou `.yaml`. O daemon indexa em
FTS5 dentro do proprio SQLite, so o que mudou, sem embedding e sem custo. O
agente busca com `knowledge_search`, que devolve cada trecho com a citacao
`[arquivo:linha]` pronta, e a resposta precisa citar.

## Specs e decisoes

`spec_write` grava em `.agent-hub/specs/` a especificacao antes de executar uma
tarefa grande. `.agent-hub/decisions/` guarda decisoes com o motivo.

## Teto

Instrucoes, glossario e memoria somados nao passam de 15% da janela do modelo. O
que ficou de fora aparece na conversa com o motivo. As ferramentas de conectores
tem teto proprio de 25% da janela.

## Ponto de retomada

Quando a conversa fica parada por 90 segundos, o daemon escreve um resumo
estruturado (tarefa, status, proximo passo, pendencias, arquivos, notas) com o
modelo local, sem custo. Ele aparece como "De onde paramos" quando voce volta.

## Versionar ou nao

`specs/` e `decisions/` sao documentacao e valem commit. `memory/` depende do seu
time. Para deixar tudo fora do git, ponha `.agent-hub/` no `.gitignore` do
projeto. Voce ve e apaga tudo em Configuracoes, Contexto.
