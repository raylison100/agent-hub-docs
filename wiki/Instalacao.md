# Instalacao

Hoje a instalacao e a partir do codigo. Um pacote unico, com o comando
`agent-hub instalar`, esta no [Roadmap](Roadmap).

## Requisitos

- Linux ou Windows com WSL2 (Ubuntu).
- Node 24 e pnpm. Com nvm: `nvm install 24 && nvm use 24 && npm i -g pnpm`.
- git.
- Docker, se for usar modelo local pelo Ollama ou gerar o app de desktop.
- Chave de API de pelo menos um provedor, ou um modelo local.

## Baixar

```bash
git clone https://github.com/raylison100/agent-hub.git
cd agent-hub
bash clonar-tudo.sh
```

O `clonar-tudo.sh` baixa os outros repositorios nas subpastas.

## Subir pela primeira vez

```bash
make build
make dev
```

Na primeira vez, o `dev.sh` cria `~/.agent-hub/config.toml` e o `.env` de
modelo, sobe o Ollama num container quando ha Docker e inicia o daemon, que
serve a interface em `http://127.0.0.1:47311`.

## Configurar

Edite `~/.agent-hub/config.toml`:

```toml
agents_dir = "/home/usuario/Projects/agent-hub/agents"
workspaces = ["/home/usuario/Projects"]
```

- `agents_dir`: pasta com perfis, precos e regras. Pode ser o `agents` clonado
  ou, melhor, uma copia sua fora de repositorio publico, porque e nela que ficam
  os seus conectores. Sem essa linha, o daemon procura em `~/.agent-hub/agents`.
- `workspaces`: pastas onde os agentes podem trabalhar. Fora delas o daemon
  recusa criar conversa.

Depois `make reiniciar`.

## Deixar o daemon sempre ligado

```bash
make servico
```

Instala um servico de usuario do systemd com reinicio automatico e liga o linger,
para ele subir junto com a maquina sem voce abrir um terminal. Os logs vao para
`~/.agent-hub/daemon.log`. No WSL, o systemd precisa estar habilitado em
`/etc/wsl.conf`. Com o servico instalado, `make reiniciar` reinicia por ele, e a
tela Configuracoes, Conexao tambem consegue reiniciar o daemon.

## Modelo local

```bash
make ollama-gpu
```

Sobe o Ollama num container com a GPU. Limites de memoria e o que esperar de
velocidade em [Modelo local](Modelo-local).

## App de desktop

```bash
make windows
make linux
```

Gera o instalador do Windows e os pacotes `.deb` e `.rpm` em
`desktop/dist-bundle`. No Windows, copie o instalador para uma pasta do Windows
antes de executar. Veja [Desktop, celular e acesso remoto](Desktop-celular-e-acesso-remoto).

## Atualizar

```bash
git pull
for d in core daemon web agents; do git -C $d pull; done
make build
make reiniciar
```

## Comandos do Makefile

`make` sem argumento lista tudo: subir e parar, servico, token, build, testes,
checagem de tipos, Ollama e empacotamento.
