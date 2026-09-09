# ADR 0003. Daemon como fonte unica da verdade

Estado: aceito. Data: 2026-09-09.

## Contexto

Desktop, navegador e celular precisam ver o mesmo estado. Opcoes:

1. Cada cliente guarda estado local e sincroniza entre si (CRDT ou servidor
   de sync).
2. Um servidor na nuvem guarda o estado e todos os clientes leem dele.
3. O daemon na maquina do usuario guarda o estado e os clientes sao
   janelas para ele.

## Decisao

Opcao 3. Sessoes, mensagens, ledger, aprovacoes e eventos vivem no SQLite
do daemon. Clientes mantem apenas cache de leitura.

## Motivos

- Elimina conflito e merge. Nao ha duas copias que possam divergir.
- O daemon ja precisa existir para executar ferramentas na maquina. Ele e o
  unico processo que tem tudo: chaves, arquivos, historico.
- Nenhum dado de codigo ou de conversa sai da maquina para um servidor de
  terceiros. O relay nao guarda nada.
- Simples de implementar: um `seq` por sessao resolve reconexao.

## Consequencias

- Se a maquina esta desligada, nao ha acesso ao historico alem do cache do
  cliente. Aceito para um produto pessoal.
- Sessoes nao migram entre daemons. Documentado como fora de escopo.
- Backup e responsabilidade do usuario: copiar o arquivo SQLite. O daemon
  oferece `export` para facilitar.
