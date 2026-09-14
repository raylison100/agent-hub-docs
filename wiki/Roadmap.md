# Roadmap

## Feito: instalar sem clonar

Pacote unico com o daemon, o core embutido, a interface e os agentes iniciais,
instalado com `npm install -g` e preparado com `agent-hub instalar`. Veja
[Instalacao](Instalacao).

## Feito: app de desktop sem daemon

Quando o app nao encontra o daemon, mostra os comandos de instalacao com botao
de copiar (no Windows, com os passos do WSL) e conecta sozinho assim que o
daemon subir.

## Feito: Release automatica

`make versao VERSAO=X.Y.Z` marca a versao nos nove repositorios, e o GitHub
Actions monta, testa e publica o pacote. Veja [Desenvolvimento](Desenvolvimento).

## Proximo

- Incluir o instalador do Windows e os pacotes `.deb` e `.rpm` na Release.

## Parado

- Respostas deterministicas para perguntas repetidas (contar commits, status de
  merge request, custo do dia) sem laco de agente.

## Avaliado e fora por enquanto

- Perfis com modelos MoE, llama-server com decodificacao especulativa e ajuste
  fino do modelo local: dependem de placa de video maior.
- Cascata ligada por padrao: medida e desligada, ver
  [Cascata e verificacao](Cascata-e-verificacao).
- Compactacao feita pelo servidor do provedor: recusada em ADR, custa caro e
  vale so para um provedor.
- Canal Slack, push por dominio HTTPS e app para macOS.
