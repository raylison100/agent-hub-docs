# Documentacao da iniciativa Agent Hub

Planejamento, arquitetura e decisoes do Agent Hub, escritos antes e durante a
construcao. Para usar o projeto, comece pela [wiki](https://github.com/raylison100/agent-hub/wiki);
as paginas dela ficam versionadas aqui em `wiki/`.

Os documentos numerados registram o desenho original e podem citar nomes de
perfil que mudaram depois; o estado atual esta na wiki e nos READMEs de cada
repositorio.

Leia em ordem. Cada documento e curto e cobre um assunto.

| Documento | Conteudo |
|-----------|----------|
| [01-visao.md](01-visao.md) | O que estamos construindo, para quem, o que fica fora |
| [02-arquitetura.md](02-arquitetura.md) | Componentes, fluxo de dados, protocolo entre partes |
| [03-repositorios.md](03-repositorios.md) | Responsabilidade, stack e interface publica de cada repositorio |
| [04-controle-de-custos.md](04-controle-de-custos.md) | Ledger, precos, orcamentos, cache por provedor, roteamento |
| [05-sincronizacao.md](05-sincronizacao.md) | Como desktop, web e mobile veem o mesmo estado |
| [06-execucao-remota.md](06-execucao-remota.md) | Como um cliente remoto constroi codigo na maquina do usuario com seguranca |
| [07-agentes.md](07-agentes.md) | Formato do perfil de agente e os tres agentes iniciais |
| [08-roadmap.md](08-roadmap.md) | Fases, entregas e criterios de aceite |
| [09-extensoes.md](09-extensoes.md) | MCP completo, skills, plugins e hooks, com compatibilidade com o Claude Code |
| [10-automacao.md](10-automacao.md) | Agendamentos, gatilhos externos, acoes em terceiros e canais de mensagem |
| [11-determinismo.md](11-determinismo.md) | O que o harness decide por regra e o que fica para o modelo |
| [12-protocolos.md](12-protocolos.md) | Padroes de mercado adotados e o que fica em formato proprio |
| [adr/](adr/) | Decisoes de arquitetura registradas com contexto e consequencias |

## Convencoes desta documentacao

- Portugues, sem emojis.
- Uma decisao de arquitetura vira um ADR em `adr/`, numerado e nunca editado
  depois de aceito. Uma mudanca gera um novo ADR que substitui o anterior.
- Termos fixos: **daemon** (servico local na maquina do usuario), **cliente**
  (desktop, navegador ou celular), **relay** (retransmissor), **perfil**
  (configuracao de um agente), **run** (uma execucao do loop de agente a partir
  de uma mensagem do usuario), **ledger** (registro de custo por chamada).

## Parte do Agent Hub

Este repositorio e uma das partes do [Agent Hub](https://github.com/raylison100/agent-hub),
um gerenciador de modelos de IA que roda na sua maquina. A documentacao geral
esta na [wiki](https://github.com/raylison100/agent-hub/wiki).

| Repositorio | Papel |
|---|---|
| [agent-hub](https://github.com/raylison100/agent-hub) | ponto de partida, Makefile, scripts e wiki |
| [agent-hub-core](https://github.com/raylison100/agent-hub-core) | biblioteca TypeScript: adaptadores, laco do agente, custo, roteamento, ferramentas, protocolo |
| [agent-hub-daemon](https://github.com/raylison100/agent-hub-daemon) | servico local: sessoes, runs, aprovacoes, automacao, conectores, API WebSocket |
| [agent-hub-web](https://github.com/raylison100/agent-hub-web) | interface Vue 3 como PWA, a mesma no navegador, no celular e no desktop |
| [agent-hub-agents](https://github.com/raylison100/agent-hub-agents) | perfis, papeis, skills, workflows, precos, roteamento e politicas, em texto |
| [agent-hub-desktop](https://github.com/raylison100/agent-hub-desktop) | app Tauri 2 para Windows e Linux |
| [agent-hub-relay](https://github.com/raylison100/agent-hub-relay) | retransmissor sem estado para acesso remoto |
| [agent-hub-channels](https://github.com/raylison100/agent-hub-channels) | clientes em plataformas de mensagem, hoje Telegram |
| [agent-hub-docs](https://github.com/raylison100/agent-hub-docs) | planejamento, arquitetura, ADRs e a fonte das paginas da wiki |

## Licenca

[PolyForm Noncommercial 1.0.0](LICENSE). Pode ler, estudar, modificar e usar
para fins pessoais, de pesquisa, ensino ou em organizacao sem fins lucrativos.
Uso comercial nao e permitido sem autorizacao do autor.

Required Notice: Copyright (c) 2026 Raylison Nunes (https://github.com/raylison100)
