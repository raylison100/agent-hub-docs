# 05. Sincronizacao entre desktop, web e mobile

## Modelo

Nao ha sincronizacao entre copias de dados. Ha um unico dado, no daemon, e
varias janelas para ele. Isso elimina conflito, merge e CRDT na primeira
versao.

```
desktop  ---- ws local ---->  daemon (SQLite)
navegador --- ws relay ---->  daemon
celular   --- ws relay ---->  daemon
```

Todos os clientes conectados recebem os mesmos eventos em tempo real. Um
cliente que abre depois pede o estado e recebe o mesmo que os outros ja
viram.

## Reconexao e catch-up

- Cada evento tem `seq` crescente por sessao, persistido no SQLite.
- O cliente guarda o ultimo `seq` visto por sessao em IndexedDB.
- Ao conectar, envia `sync {session_id, since_seq}`. O daemon responde com os
  eventos faltantes, em ordem, e depois passa ao tempo real.
- Runs em andamento continuam no daemon mesmo com todos os clientes
  desconectados. Reconectar mostra o progresso.

## Leitura offline

O cliente mantem em IndexedDB a lista de sessoes e as ultimas mensagens
lidas. Sem conexao, o usuario le o historico. Enviar mensagem exige conexao;
o cliente mostra o estado do dispositivo (online, offline, ocupado).

## Varios dispositivos alvo

Um usuario pode ter mais de um daemon (PC do trabalho, PC de casa, servidor).
Cada daemon se registra no relay com `device_id` e nome. O cliente escolhe o
dispositivo antes de criar a sessao. A sessao pertence ao daemon onde foi
criada.

Fora de escopo na primeira versao: mover ou espelhar sessao entre daemons.
Se virar necessidade, o caminho e exportar e importar sessao via ledger e
mensagens, sem merge.

## O que sincroniza por git, nao pelo daemon

Perfis de agente, precos, MCP e politicas vivem no repositorio `agents`.
Cada daemon aponta para um clone local. Atualizar e `git pull`. O daemon
observa o diretorio e recarrega perfis sem reiniciar.

## Autenticacao

- **Conta.** Um `account_token` longo, gerado uma vez, guardado no cliente. O
  relay o usa para saber quais dispositivos aquele cliente pode ver.
- **Dispositivo.** Cada daemon tem um `device_secret` gerado no `pair`. O
  relay valida o registro com ele.
- **Emparelhamento.** `agent-hub-daemon pair` imprime um QR code com
  `relay_url + account_token`. O celular le e passa a ver aquele daemon.
- **Local.** Desktop conectado em `127.0.0.1` usa um token de arquivo com
  permissao restrita ao usuario. Nenhuma porta e aberta para a rede por
  padrao.

Tokens rotacionam pelo CLI. Tokens revogados derrubam conexoes na hora.

## Transporte

- Relay apenas por TLS.
- Fase 3 entrega o relay em texto claro por dentro do TLS. Fase 4 avalia
  cifrar os quadros de ponta a ponta com segredo derivado do emparelhamento,
  para que nem o relay veja o conteudo. O protocolo ja reserva um campo
  `enc` no quadro para isso.

## Alternativa sem relay

Com Tailscale ou WireGuard, o celular alcanca `daemon:47311` direto. O
cliente aceita URL de daemon manual. O relay vira opcional. Ver `adr/0004`.
