# Roteamento e custo

Com o agente em **Auto**, cada mensagem passa por esta sequencia, configurada em
`routing.json`:

1. **Regras.** Casam por intencao, palavras, arquivos, pasta ou tamanho do pedido
   e fixam um agente. Exemplo: revisao e arquitetura vao para o `claude`.
2. **Intencao por palavra-chave.** `revisar`, `explicar`, `implementar`,
   `arquitetura`, cada uma com sua lista.
3. **Classificador.** Se nenhuma palavra casou, um modelo barato escolhe a
   intencao entre as declaradas.
4. **Pontuacao.** Os agentes que sobram recebem uma nota:
   `capacidade - peso_custo x custo_normalizado - peso_contexto x pressao_de_janela`.
   Ganha a maior.

## Exclusoes antes da pontuacao

Um agente sai da disputa quando:

- o pedido tem imagem e o agente nao tem visao;
- o pedido fala em delegar e o agente nao delega;
- o contexto da conversa nao cabe na janela dele, com margem;
- a classe de latencia nao combina (`lote` nao atende conversa);
- a capacidade para a intencao fica abaixo do minimo;
- o preco nao esta na tabela.

## O que entra no custo

O custo estimado usa o historico real que vai para o modelo mais o pedido novo,
nao so o texto da mensagem. Uma conversa longa encarece mais o modelo caro e
empurra para quem tem janela e preco melhores.

```json
"scoring": {
  "cost_weight": 0.5,
  "batch_cost_weight": 0.85,
  "context_weight": 0.2,
  "min_capability": 0.4,
  "context_margin": 1.5
}
```

`batch_cost_weight` vale para agendamento, gatilho e workflow: sem ninguem
esperando, o custo pesa mais.

## Aprendizado pelo voto

Os votos bom e ruim de cada agente em cada intencao ajustam a capacidade usada na
pontuacao. `agent-hub-daemon feedback` mostra o ajuste; `agent-hub-daemon route
"texto"` mostra a decisao e o ranking sem gastar tokens.

## Papel escolhido pelo pedido

Quando a sessao nao tem papel, `roles` no `routing.json` escolhe um pelo texto do
pedido. A primeira regra que casa vale, e `improve: false` desliga o melhorador
de prompt para aquele pedido, quando a skill do papel ja faz esse trabalho:

```json
"roles": [
  { "when": { "keywords": ["arte", "stories", "campanha"] }, "role": "marketing", "improve": false }
]
```

Prefira palavras que nao aparecem em pedido de codigo. O aviso de roteamento na
conversa diz quando o papel veio da regra.

## Melhorador de prompt

`prompt_improver` reescreve o pedido para o agente escolhido, sem inventar fato.
Pedido curto vai para o modelo local; pedido grande, para o remoto.

## Ledger e orcamentos

Cada chamada ao modelo grava: agente, provedor, modelo, tokens de entrada, saida,
cache lido, cache escrito e raciocinio, custo, versao da tabela de precos,
latencia e motivo de parada. Chamada sem uso informado pelo provedor fica marcada
como incompleta, nunca estimada.

Orcamentos sao conferidos antes de cada chamada, por run, sessao, agente por dia,
automacao por mes e global por mes. Com 80% gasto aparece aviso; no limite, o run
para. Da para liberar mais para um run especifico pela interface.

## Economia de cache

O comeco do prompt nao muda entre mensagens da mesma conversa: as ferramentas
sao escolhidas uma vez por sessao e a memoria do projeto entra na mensagem que a
ativou. Assim o provedor cobra o historico como cache lido, muito mais barato, e
o Ollama nao rele a conversa inteira. Medido no qwen3 local: o mesmo turno caiu
de 2,7 s para 0,7 s de leitura do prompt.
