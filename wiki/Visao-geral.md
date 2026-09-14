# Visao geral

## O problema

Quem usa IA para programar costuma ficar preso a um provedor, nao sabe quanto
cada tarefa custou e depende de um aplicativo por dispositivo. E paga preco de
modelo de fronteira ate para perguntas que um modelo barato ou local resolveria.

## O que o Agent Hub faz diferente

**Escolhe o modelo por mensagem.** Nao existe "o modelo da conversa". Cada
pedido passa por regras, por um classificador de intencao e por uma pontuacao
que pesa capacidade, preco e o tamanho do contexto. Uma revisao de merge request
pode ir para o Claude e a pergunta seguinte, na mesma conversa, para o DeepSeek.

**Trata custo como parte do produto.** Toda chamada entra num ledger com tokens
de entrada, saida, cache e raciocinio, preco da tabela e latencia. Orcamentos
param o run antes de estourar. O desconto de horario do DeepSeek entra na conta.

**Decide por codigo o que nao deve ficar com o modelo.** Validacao de chamada de
ferramenta com JSON Schema, politica de risco, comando destrutivo sempre pedindo
aprovacao, fases por perfil, workflows com etapas fixas e verificacao da
resposta por codigo.

**Roda na sua maquina.** O daemon guarda tudo localmente e executa as
ferramentas nos seus projetos. As chaves de API ficam cifradas nele. A mesma
conversa aparece no navegador, no app de desktop, no celular e no Telegram.

**Aproveita o que ja existe.** Le skills, plugins, hooks e servidores MCP no
formato do Claude Code. Fala MCP, A2A, OAuth 2.1, Standard Webhooks e
OpenTelemetry.

## Para quem

Desenvolvedor que trabalha em varios projetos, usa mais de um provedor, quer
saber quanto cada tarefa custou e quer disparar trabalho na propria maquina de
qualquer lugar. O produto e pessoal: uma pessoa, varios dispositivos.

## Fora de escopo

- Multiusuario e times.
- Sincronizar conversas entre dois daemons diferentes.
- Loja de plugins.
- Treinar modelos.

## Principios

- **O modelo propoe, o harness dispoe.**
- **Uma fonte da verdade:** o daemon.
- **Modelo local nao e modelo confiavel:** a politica de seguranca vale igual.
- **Configuracao e texto:** perfis, precos e regras em arquivos versionaveis.
- **Medir antes de ligar:** recursos que dependem do modelo sao medidos com
  perguntas reais, e o que nao compensa fica desligado, com o numero registrado.
