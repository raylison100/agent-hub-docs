# Desktop, celular e acesso remoto

Todo cliente e uma janela para o mesmo daemon. Conversas, aprovacoes e custo
sao os mesmos em qualquer um.

## Navegador

`http://127.0.0.1:47311`, servido pelo proprio daemon. Na mesma maquina conecta
sozinho.

## App de desktop

Tauri 2, com bandeja e janela que esconde ao fechar.

- **Windows**: o app conecta no daemon que roda no WSL. Gere com `make windows`,
  copie o instalador de `desktop/dist-bundle` para uma pasta do Windows e rode.
  Executado direto de `\\wsl.localhost\...`, o instalador falha.
- **Linux**: `make linux` gera `.deb` e `.rpm`. Sem daemon no ar, o app sobe um.

O seletor de pasta usa a janela nativa do sistema e traduz caminhos do Windows
para o WSL.

## Celular

A interface e um PWA: abra pelo navegador do celular e instale na tela inicial.
Aprovacoes pendentes e fim de run chegam como notificacao push.

## De fora de casa

### Com senha

1. Defina a senha em Configuracoes, Conexao, ou pelo terminal:

   ```bash
   echo -n "sua senha" | agent-hub-daemon senha
   ```

2. No outro dispositivo, entre com a senha uma vez. Ele recebe uma credencial
   propria, que fica guardada so nele.
3. Na mesma tela voce ve cada dispositivo com nome e ultimo acesso e revoga um
   sem mexer nos outros.

A senha e guardada com scrypt; a credencial do dispositivo, so como hash. Tres
erros seguidos e a origem espera, com a espera crescendo a cada tentativa.

### Pelo relay

Para chegar ao daemon sem abrir porta no roteador:

1. Suba o [relay](https://github.com/raylison100/agent-hub-relay) num servidor,
   atras de TLS.
2. No `config.toml` do daemon, `relay_url = "wss://seu-relay"`.
3. `agent-hub-daemon pair` mostra o token de conta, o id do dispositivo, o link e
   o QR de emparelhamento.

O daemon abre a conexao de saida. O conteudo vai cifrado ponta a ponta entre o
cliente e o daemon; o relay so encaminha.

Uma VPN como Tailscale tambem funciona: aponte o cliente para o endereco do
daemon na VPN.

## Telegram

O [channels](https://github.com/raylison100/agent-hub-channels) liga um bot:
mensagem vira pedido, aprovacao vira botao, e so os ids que voce liberar sao
atendidos.
