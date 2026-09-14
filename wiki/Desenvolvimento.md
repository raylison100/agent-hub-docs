# Desenvolvimento

## Ambiente

- Linux ou WSL, Node 24 (com nvm: `nvm use 24`) e pnpm.
- `core`, `daemon`, `relay` e `channels` usam TypeScript 7; `web` fixa
  TypeScript 5 por causa do `vue-tsc`.
- O `daemon` consome o `core` por `link:../core`: compile o core antes.
- Docker para Ollama e para gerar o desktop.

## Organizacao

Cada parte e um repositorio git proprio, clonado como subpasta do
`agent-hub`. O `clonar-tudo.sh` da raiz baixa todos.

## Comandos

```bash
make build     # core, daemon e web
make teste     # testes do core
make tipos     # typecheck de core, daemon, relay, channels e web
make dev       # sobe tudo
make reiniciar # reinicia o daemon, pelo systemd quando instalado
```

Durante o desenvolvimento do daemon, `pnpm dev <comando>` roda direto do fonte.

## Testes

O core tem a suite com vitest: provedores com fixtures, precos, orcamentos,
perfis, roteamento, pontuacao, selecao de ferramentas, contexto do projeto,
verificacao, workflows, OAuth e o runner com adaptador falso.

Recurso que depende de modelo e validado tambem com run real contra o daemon, e
o numero medido fica registrado na documentacao.

## Convencoes

- Portugues no codigo, na documentacao e nos commits.
- Sem emojis em nenhum lugar.
- Sem comentarios dentro do corpo das funcoes; no maximo um docblock curto dizendo
  o que a funcao faz. O motivo vai na mensagem de commit.
- Commits por bloco de mudanca, mensagem com o que mudou e por que, e com numero
  medido quando for otimizacao.
- Decisao de arquitetura vira ADR em `agent-hub-docs/adr`, nunca editado depois
  de aceito.

## Wiki

As paginas da wiki ficam em `agent-hub-docs/wiki`. Para publicar, copie para o
repositorio da wiki do `agent-hub` (`agent-hub.wiki.git`) e faca push.

## Contribuir

Issues e pull requests sao bem-vindos para uso nao comercial, dentro da
[Licenca](Licenca). Ao contribuir, voce concorda que a contribuicao segue a mesma
licenca.
