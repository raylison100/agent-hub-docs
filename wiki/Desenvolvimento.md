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

## Versoes e Release

O Agent Hub tem **um numero de versao para o produto inteiro**, igual em todos os
repositorios, porque o pacote junta daemon, core, interface e agentes. A versao
segue o versionamento semantico: o terceiro numero sobe em correcao, o segundo
em funcionalidade nova, o primeiro em mudanca que quebra compatibilidade
(enquanto for `0.x`, a quebra pode vir no segundo).

Para lancar uma versao:

```bash
make versao VERSAO=0.3.0
```

O `scripts/versao.sh`:

1. confere que os nove repositorios estao na `main`, sem mudanca pendente e
   iguais ao GitHub, e que a tag ainda nao existe;
2. troca a versao nos `package.json`, no `tauri.conf.json`, no `Cargo.toml` e no
   `Cargo.lock`;
3. commita "Versao X.Y.Z" onde mudou e cria a tag `vX.Y.Z` nos nove;
4. envia os oito repositorios e, por ultimo, a raiz.

A tag da raiz dispara o workflow **Release** (`.github/workflows/release.yml`),
que clona os outros repositorios na mesma tag e roda tres jobs em paralelo: o
pacote (compila, roda os testes do core, monta e instala num container limpo),
o app de Linux (`.deb` e `.rpm`) e o app de Windows (instalador NSIS, em runner
Windows). Com os tres verdes, publica a Release com tudo, uma copia
`agent-hub.tgz` de nome fixo e as notas geradas pelos commits de cada
repositorio desde a versao anterior. Um push na branch `ci/release` roda os
tres jobs sem publicar, para testar mudancas no workflow. O
workflow usa o token do proprio GitHub Actions: ninguem precisa de token
pessoal nem do `gh`.

## Wiki

As paginas da wiki ficam em `agent-hub-docs/wiki`. Para publicar, copie para o
repositorio da wiki do `agent-hub` (`agent-hub.wiki.git`) e faca push.

## Contribuir

Issues e pull requests sao bem-vindos para uso nao comercial, dentro da
[Licenca](Licenca). Ao contribuir, voce concorda que a contribuicao segue a mesma
licenca.
