# 01. Visao

## Problema

Hoje cada ferramenta de agente prende o usuario a um provedor, esconde o
custo real de cada tarefa e roda em um unico dispositivo. Quem usa varios
modelos (Claude, DeepSeek, OpenAI, modelo local) precisa trocar de aplicativo,
perde o historico e nao consegue comparar custo por tarefa.

## Objetivo

Um unico cliente de agentes que:

1. Usa qualquer provedor por agente: Claude, DeepSeek, OpenAI, e modelos
   locais na maquina ou na rede (Ollama, vLLM, LM Studio, llama.cpp).
2. Mostra e limita custo em tempo real, por chamada, run, sessao, agente e dia.
3. Roda em desktop (Windows, macOS, Linux), navegador e celular, sempre
   olhando para o mesmo estado.
4. Executa codigo e ferramentas na maquina do usuario, mesmo quando o comando
   vem do celular ou de outro computador.
5. Permite criar agentes por tarefa em arquivos de texto versionaveis.

## Principios

- **Custo e cidadao de primeira classe.** Nenhuma chamada de modelo acontece
  sem passar pelo ledger e pelos orcamentos. A interface mostra custo antes de
  virar surpresa.
- **Uma fonte da verdade.** O daemon na maquina do usuario guarda sessoes,
  mensagens e ledger. Clientes sao janelas para esse estado.
- **Modelo local nao e modelo confiavel.** Aprovacao e sandbox valem para
  qualquer modelo. A origem do modelo nao muda a politica de seguranca.
- **Perfis sao texto.** Agentes, precos e configuracao de MCP vivem em
  arquivos, em repositorio proprio, para sincronizar entre maquinas com git.
- **Reaproveitar o que ja existe.** SDKs oficiais dos provedores, SDK oficial
  de MCP, Tauri, Vue. Codigo proprio so onde da controle que a biblioteca
  nao da.

## Fora de escopo (por enquanto)

- Multiusuario e times. O produto e pessoal. Uma conta, varios dispositivos.
- Sincronizar sessoes entre dois daemons diferentes (trabalho e casa). Cada
  sessao vive no daemon onde nasceu. Ver `05-sincronizacao.md`.
- Loja de plugins ou marketplace de agentes.
- Fine-tuning ou treinamento de modelos.
- Interface de terminal. O foco e grafico. Um CLI minimo existe so para
  desenvolver e testar o daemon.

## Usuario alvo

Desenvolvedor que trabalha em varios projetos e empresas, usa varios
provedores, quer saber exatamente quanto cada tarefa custou e quer disparar
trabalho na propria maquina de qualquer lugar.
