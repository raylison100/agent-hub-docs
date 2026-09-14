# Roadmap

## Feito: instalar sem clonar

Pacote unico com o daemon, o core embutido, a interface e os agentes iniciais,
instalado com `npm install -g` e preparado com `agent-hub instalar`. Veja
[Instalacao](Instalacao).

## Proximo

- O app de desktop no Windows mostrar o comando de instalar no WSL quando nao
  encontrar o daemon.
- Publicar cada versao do pacote como Release automaticamente.

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
