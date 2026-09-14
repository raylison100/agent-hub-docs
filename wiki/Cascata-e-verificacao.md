# Cascata e verificacao

A ideia: o modelo local tenta primeiro, o **codigo** confere a resposta, e so vai
para um modelo pago se a conferencia falhar. Voce paga nuvem so nas falhas.

## Verificacao por codigo

`verifyRun` confere o resultado de um run sem chamar modelo nenhum:

| Verificacao | Recusa quando |
|---|---|
| `parada` | o run nao terminou normalmente |
| `resposta` | veio vazia, comecou com raciocinio vazado ou diz que nao encontrou |
| `fundamentacao` | nao leu nada do workspace, ou cita arquivo que nao leu neste run |
| `citacoes` | cita arquivo que nao existe, linha alem do fim, ou `[arquivo:linha]` literal |
| `ferramentas` | todas as chamadas de ferramenta falharam |

## Ligar

No `routing.json`:

```json
"cascade": {
  "agent": "qwen3",
  "escalate_to": "deepseek",
  "intents": ["explicar"]
}
```

Vale com agente automatico, sem papel, sem imagem e com a conversa cabendo no
modelo local. Quando a verificacao recusa, o pedido vai para o agente que o
roteador escolheu, a conversa mostra o motivo, e a tentativa recusada fica
guardada como mensagem filha, fora do historico.

## Por que vem desligada

Medido com 12 perguntas sobre o proprio codigo do projeto, no qwen3:8b com 8k de
contexto:

| | Primeira versao | Com fundamentacao | So DeepSeek |
|---|---|---|---|
| aceitas no local | 11 de 12 | 3 de 12 | - |
| aceitas e erradas | 8 | 1 | - |
| custo das 12 | 0,001 USD | 0,019 USD | 0,035 USD |
| tempo das 12 | 136 s | 216 s | 177 s |

A primeira versao so pegava citacao errada, e o modelo local respondia sem abrir
arquivo nenhum. Com a checagem de fundamentacao, a resposta errada que passou foi
de um run que leu o arquivo errado e citou esse arquivo corretamente: nenhuma
checagem por codigo pega isso sem entender a pergunta.

Nesse cenario a cascata economiza centavos, demora mais e deixa passar uma
resposta errada em doze. Vale ligar com um modelo local maior ou para uma
intencao em que o local acerte quase sempre.

## Fallback de ferramenta

Independente da cascata, o agente com `fallback_agent` escala quando erra a
chamada de ferramenta mais vezes que `repair_attempts`. A tentativa que falhou
tambem vira mensagem filha, e o custo das duas entra no run.
