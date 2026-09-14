# Instalacao

Dois caminhos: o **pacote**, para usar, e o **codigo**, para desenvolver.

## Pelo pacote

Nao precisa clonar nada. Um arquivo traz o daemon, a interface e os agentes
iniciais.

### Requisitos

- Linux ou Windows com WSL2 (Ubuntu).
- Node 22 ou superior.
- Ferramentas de compilacao, porque o terminal do daemon (`node-pty`) compila na
  instalacao: `sudo apt install -y build-essential python3`.
- systemd de usuario para o daemon ficar sempre ligado. No WSL, habilite em
  `/etc/wsl.conf` com `[boot]` e `systemd=true`.

### Instalar

```bash
npm install -g https://github.com/raylison100/agent-hub/releases/download/v0.2.0/agent-hub-0.2.0.tgz
agent-hub instalar
```

O `agent-hub instalar`, em ordem:

1. confere se os modulos nativos carregam;
2. cria `~/.agent-hub/config.toml`, liberando `~/Projects` (ou a pasta pessoal)
   para os agentes;
3. copia os agentes iniciais para `~/.agent-hub/agents`;
4. grava e liga o servico do systemd, com linger para subir junto com a maquina;
5. espera o daemon responder e mostra o endereco.

Rodar de novo nao sobrescreve config nem agentes. Se ja existir um servico
`agent-hub` apontando para outra instalacao, ele para e pede
`--substituir-servico`. Sem systemd, use `agent-hub instalar --sem-servico` e
depois `agent-hub start`.

Abra `http://127.0.0.1:47311` e siga em [Primeiros passos](Primeiros-passos).

### Atualizar

```bash
agent-hub atualizar https://github.com/raylison100/agent-hub/releases/download/vX.Y.Z/agent-hub-X.Y.Z.tgz
```

Instala a versao nova e regrava o servico com o Node atual, o que tambem corrige
o servico depois de trocar a versao do Node pelo nvm. Sem argumento,
`agent-hub atualizar` so regrava o servico e reinicia.

## Pelo codigo

### Requisitos

- Linux ou Windows com WSL2 (Ubuntu).
- Node 24 e pnpm. Com nvm: `nvm install 24 && nvm use 24 && npm i -g pnpm`.
- git.
- Docker, se for usar modelo local pelo Ollama ou gerar o app de desktop.
- Chave de API de pelo menos um provedor, ou um modelo local.

### Baixar

```bash
git clone https://github.com/raylison100/agent-hub.git
cd agent-hub
bash clonar-tudo.sh
```

O `clonar-tudo.sh` baixa os outros repositorios nas subpastas.

### Subir pela primeira vez

```bash
make build
make dev
```

Na primeira vez, o `dev.sh` cria `~/.agent-hub/config.toml` e o `.env` de
modelo, sobe o Ollama num container quando ha Docker e inicia o daemon, que
serve a interface em `http://127.0.0.1:47311`.

### Configurar

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

### Deixar o daemon sempre ligado

```bash
make servico
```

Instala um servico de usuario do systemd com reinicio automatico e liga o linger,
para ele subir junto com a maquina sem voce abrir um terminal. Os logs vao para
`~/.agent-hub/daemon.log`. No WSL, o systemd precisa estar habilitado em
`/etc/wsl.conf`. Com o servico instalado, `make reiniciar` reinicia por ele, e a
tela Configuracoes, Conexao tambem consegue reiniciar o daemon.

### Modelo local

```bash
make ollama-gpu
```

Sobe o Ollama num container com a GPU. Limites de memoria e o que esperar de
velocidade em [Modelo local](Modelo-local).

### App de desktop

```bash
make windows
make linux
```

Gera o instalador do Windows e os pacotes `.deb` e `.rpm` em
`desktop/dist-bundle`. No Windows, copie o instalador para uma pasta do Windows
antes de executar. Veja [Desktop, celular e acesso remoto](Desktop-celular-e-acesso-remoto).

### Atualizar

```bash
git pull
for d in core daemon web agents; do git -C $d pull; done
make build
make reiniciar
```

### Gerar o pacote

```bash
make pacote
make pacote-testar
```

`make pacote` gera `dist-pacote/agent-hub-VERSAO.tgz` com o daemon, o core
embutido, a interface e o modelo de agentes (sem agendamentos e sem `mcp.json`).
`make pacote-testar` instala esse arquivo num container Node limpo, sem nenhum
repositorio, e confere instalacao, agentes, interface e saude.

### Comandos do Makefile

`make` sem argumento lista tudo: subir e parar, servico, token, build, testes,
checagem de tipos, Ollama e empacotamento.
