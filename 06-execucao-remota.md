# 06. Execucao remota na maquina do usuario

O objetivo e disparar do celular ou do navegador uma tarefa que le, escreve
e roda codigo na maquina onde o daemon esta. A mesma politica vale quando o
comando vem do desktop local. A origem do cliente nao relaxa nada.

## Ferramentas nativas

Executadas pelo daemon, nunca pelo cliente.

| Ferramenta | Argumentos | Risco |
|------------|------------|-------|
| `list_dir` | caminho | leitura |
| `read_file` | caminho, faixa de linhas | leitura |
| `search` | padrao, caminho, glob | leitura |
| `write_file` | caminho, conteudo | escrita |
| `edit_file` | caminho, trecho antigo, trecho novo | escrita |
| `run_command` | comando, cwd, timeout | execucao |
| `git` | subcomando restrito (status, diff, log, add, commit, checkout de branch nova) | escrita |

Ferramentas MCP entram pelo mesmo registro e recebem classificacao de risco
no `agents/mcp.json` (`read`, `write`, `exec`).

## Workspaces

Cada sessao nasce amarrada a um workspace: um diretorio raiz permitido.
Toda ferramenta de arquivo resolve o caminho e recusa qualquer coisa fora da
raiz, inclusive links simbolicos que apontem para fora. Comandos rodam com
`cwd` dentro do workspace.

Lista de workspaces permitidos fica em `~/.agent-hub/config.toml`. Criar
sessao em diretorio fora da lista falha.

## Politica de aprovacao

Por perfil, em `agents/policies/*.json`, com tres decisoes por classe de
risco:

| Decisao | Efeito |
|---------|--------|
| `allow` | executa sem perguntar |
| `ask` | emite `approval.required` e espera o cliente |
| `deny` | recusa e devolve erro ao agente |

Padrao para todo perfil novo: leitura `allow`, escrita `ask`, execucao
`ask`. Um perfil so pode relaxar para `allow` em escrita ou execucao se
tambem declarar `workspace.allowlist` explicito.

Aprovacao pode ser lembrada por sessao (`allow_always` para aquele comando
exato ou aquele caminho). Nunca e lembrada entre sessoes.

Regras fixas que nenhum perfil sobrescreve:

- `run_command` com padroes destrutivos conhecidos (`rm -rf /`, `git push
  --force` em `main`, `DROP`, formatacao de disco) e sempre `ask`, mesmo em
  perfil `allow`.
- Comandos tem timeout padrao de 120 s e maximo de 600 s.
- Saida de comando e truncada em 50 KB antes de voltar ao modelo. O restante
  fica em arquivo de log referenciado no resultado.
- Escrita fora do workspace e `deny` sempre.

## Aprovacao a partir do celular

`approval.required` carrega ferramenta, argumentos resumidos, diff quando for
escrita, e um `expires_at`. O cliente mostra um cartao com aprovar e negar.
Sem resposta ate expirar, o daemon nega e o run continua com o erro,
deixando o agente decidir o que fazer. Expiracao padrao de 10 minutos, com
notificacao push do PWA quando suportado.

## Sandbox

Fase 1: sem sandbox, apenas politica e workspace. Suficiente para uso
pessoal com aprovacao ligada.

Fase 4: opcao de rodar `run_command` dentro de container (Docker ou Podman)
com o workspace montado, para perfis com `allow` em execucao. Sem container
disponivel, o perfil cai para `ask`.

## Auditoria

Toda chamada de ferramenta grava em `tool_events`: quem pediu (agente, run),
o que rodou, quem aprovou (dispositivo), quanto tempo levou e o resultado.
A interface mostra o historico por sessao. Exportavel em JSON.

## Segredos

- Chaves de provedor ficam em variaveis de ambiente do daemon ou no gerenciador
  de segredos do sistema (Windows Credential Manager, Keychain, Secret
  Service). Nunca em `config.toml`.
- O daemon filtra a saida de comandos e o conteudo de arquivos contra
  padroes de segredo conhecidos antes de mandar ao modelo, substituindo por
  `[redacted]`. Lista de padroes em `agents/policies/secrets.json`.
- O cliente nunca recebe chaves.
