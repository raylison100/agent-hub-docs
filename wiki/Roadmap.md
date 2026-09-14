# Roadmap

## Proximo: instalar sem clonar

Hoje e preciso clonar os repositorios e compilar. O plano:

1. O `core` passa a ir compilado dentro do pacote do daemon, e a interface
   resolvida a partir do pacote.
2. Um conjunto limpo de agentes vai dentro do pacote e e copiado uma vez para
   `~/.agent-hub/agents`.
3. Comandos `agent-hub instalar` (config, agentes iniciais, servico do systemd,
   checagem de dependencias nativas, teste de saude) e `agent-hub atualizar`.
4. `make pacote` gera o `.tgz`, distribuido por Release do GitHub.
5. Teste de instalacao em pasta limpa, sem nenhum repositorio clonado.

Depois disso, o app de desktop no Windows pode mostrar o comando de instalar no
WSL quando nao encontrar o daemon.

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
