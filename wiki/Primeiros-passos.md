# Primeiros passos

## 1. Abrir a interface

Com o daemon no ar, abra `http://127.0.0.1:47311`. Na propria maquina a conexao
e automatica: o daemon entrega a credencial so para quem chama do loopback e da
propria interface. Nao ha token para copiar.

## 2. Cadastrar uma chave

Configuracoes, **Chaves**. Cole a chave de pelo menos um provedor. Ela fica
cifrada no SQLite do daemon e nunca vai para o navegador de volta.

| Provedor | Agente que usa |
|---|---|
| Anthropic | `claude` |
| DeepSeek | `deepseek` |
| Google Gemini | `gemini` |
| OpenAI | `openai` |
| nenhuma, Ollama local | `qwen3` |

## 3. Comecar uma conversa

Na tela inicial:

- **Pasta**: o projeto onde o agente vai trabalhar (precisa estar dentro de
  `workspaces` no `config.toml`).
- **Agente**: deixe em **Auto** para o roteador escolher a cada mensagem, ou
  fixe um.
- **Papel**: opcional; `revisor` e `arquiteto` trazem prompt, ferramentas e
  politica proprios e rodam no modelo que o roteador escolher entre os
  permitidos.
- **Modo**: `Manual` pede aprovacao conforme a politica; `Aceitar edicoes`
  libera escrita e ainda pergunta antes de executar; `Planejar` so le e propoe;
  `Automatico` aprova tudo, menos comando destrutivo.
- **Melhorar prompt**: reescreve o pedido antes de mandar, com modelo local para
  pedido curto e remoto para pedido grande.
- **Esforco**: quanto o modelo pode raciocinar.

Mande a mensagem. Na conversa voce ve qual agente foi escolhido e por que, cada
ferramenta chamada, as aprovacoes e o custo da resposta.

## 4. Votar

Cada resposta tem **bom** e **ruim**. O voto ajusta a capacidade do agente para
aquela intencao no roteador. Sem voto, o roteador nao aprende.

## 5. Acompanhar o custo

Configuracoes, **Custos**: gasto por agente, modelo, conversa ou dia, taxa de
cache e exportacao CSV. O orcamento de cada agente esta em
`agents/policies/budgets.json`.

## Proximos passos

- [Agentes, perfis e papeis](Agentes-perfis-e-papeis) para ajustar os agentes.
- [Contexto do projeto e memoria](Contexto-do-projeto-e-memoria) para o agente
  conhecer seu projeto.
- [Conectores MCP](Conectores-MCP) para GitHub, Jira e outros.
