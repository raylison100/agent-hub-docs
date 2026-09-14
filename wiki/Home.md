# Agent Hub

O Agent Hub e um gerenciador de modelos de IA que roda na sua maquina. Voce
conversa com ele como conversaria com qualquer assistente de programacao, mas
por tras de cada mensagem ele decide:

- **qual modelo responde**, entre Claude, DeepSeek, Gemini, OpenAI e modelos
  locais, pesando o que cada um sabe fazer contra quanto custa;
- **o que o modelo pode fazer**, com politica de leitura, escrita e execucao e
  aprovacao para o que for arriscado;
- **quanto pode gastar**, com orcamento por run, conversa, agente e mes;
- **o que o modelo precisa saber**, juntando instrucoes, memoria e documentos do
  projeto sem estourar a janela de contexto.

A regra que guia o projeto: **o modelo propoe, o harness dispoe.** Tudo que da
para decidir por codigo e decidido por codigo.

## Por onde comecar

| Quero | Pagina |
|---|---|
| entender o que e e como funciona | [Visao geral](Visao-geral), [Arquitetura](Arquitetura) |
| instalar e rodar | [Instalacao](Instalacao), [Primeiros passos](Primeiros-passos) |
| configurar agentes e modelos | [Agentes, perfis e papeis](Agentes-perfis-e-papeis), [Roteamento e custo](Roteamento-e-custo) |
| usar modelo local | [Modelo local](Modelo-local), [Cascata e verificacao](Cascata-e-verificacao) |
| dar contexto do meu projeto | [Contexto do projeto e memoria](Contexto-do-projeto-e-memoria) |
| ligar GitHub, Jira e outros | [Conectores MCP](Conectores-MCP) |
| rodar tarefas sozinho | [Automacao](Automacao) |
| usar do celular ou de outro PC | [Desktop, celular e acesso remoto](Desktop-celular-e-acesso-remoto) |
| usar o modelo local de alguem de confianca | [Compartilhar modelos](Compartilhar-modelos) |
| saber o que e seguro | [Seguranca](Seguranca) |
| integrar com outras ferramentas | [Protocolo e integracoes](Protocolo-e-integracoes) |
| contribuir ou mexer no codigo | [Desenvolvimento](Desenvolvimento) |

## Licenca

Codigo aberto para leitura e uso nao comercial, sob a PolyForm Noncommercial
1.0.0. Veja [Licenca](Licenca).
