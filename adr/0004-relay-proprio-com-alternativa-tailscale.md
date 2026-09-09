# ADR 0004. Relay proprio sem estado, com Tailscale como alternativa

Estado: aceito. Data: 2026-09-09.

## Contexto

O celular e o navegador fora da rede local precisam alcancar o daemon.
Opcoes:

1. Expor a porta do daemon na internet.
2. VPN de malha (Tailscale, WireGuard) e conectar direto.
3. Relay proprio: o daemon abre conexao de saida, clientes conectam no
   relay, o relay encaminha.
4. Servico de tunel de terceiros (Cloudflare Tunnel, ngrok).

## Decisao

Opcao 3 como caminho padrao, opcao 2 como alternativa suportada pelo mesmo
protocolo. A opcao 1 e proibida por padrao no daemon.

## Motivos

- O relay e um servico de dezenas de linhas, sem banco, facil de subir em
  qualquer VPS. Custo proximo de zero.
- Nao exige instalar VPN no celular, o que facilita o uso do PWA.
- O daemon so faz conexao de saida. Nenhuma porta aberta na maquina do
  usuario.
- Tailscale resolve o mesmo problema sem servidor proprio. Para quem ja usa,
  e melhor. O cliente aceita URL manual do daemon por isso.

## Consequencias

- O relay ve os quadros em texto claro dentro do TLS na fase 3. Cifra de
  ponta a ponta fica para a fase 4, e o protocolo reserva o campo `enc`.
- O relay precisa de limite de taxa e expiracao de conexao ociosa para nao
  virar alvo.
- Duas formas de conexao para manter e testar: via relay e direta.
