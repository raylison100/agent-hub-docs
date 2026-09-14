# Modelo local

O perfil `qwen3` roda no Ollama, na sua maquina, sem custo por token. Serve para
classificar pedidos, reescrever prompt, resumir conversa, escrever o ponto de
retomada e para agendamentos sem pressa.

## Subir

```bash
make ollama-gpu
```

O container usa a GPU pelo NVIDIA Container Toolkit, com contexto de 8k, KV cache
em `q8_0` e flash attention. O daemon fala com ele por `OLLAMA_BASE_URL`.

## Por que a placa de video manda

Pense na memoria da placa (VRAM) como uma mesa:

- os **pesos do modelo** sao os livros: ocupam espaco fixo;
- o **contexto** sao as folhas espalhadas: cada token precisa de um pedaco da
  mesa, o KV cache;
- quando a mesa enche, parte vai para a RAM, que fica longe, e tudo fica lento.

A janela que cabe e o que sobra de mesa depois dos livros.

| Modelo | Pesos em Q4 | Contexto por mil tokens (KV q8_0) |
|---|---|---|
| qwen3:8b | ~4,9 GiB | ~75 MiB |
| qwen3:14b | ~9,0 GiB | ~83 MiB |
| qwen3:32b | ~18,8 GiB | ~133 MiB |

Some uns 0,8 GiB de overhead. Numa placa de 6 GB, o 8b com 8k de contexto ja nao
cabe inteiro e parte vai para a CPU: medimos ~13,5 tokens/s. Numa de 16 GB, o 14b
com 32k cabe inteiro.

Mais RAM de sistema nao acelera modelo que cabe na placa. O caso em que ela ajuda
e modelo MoE com especialistas na RAM.

## Os outros limites

- **Janela de treino**: o qwen3 foi treinado para ~32 mil tokens. Da para
  esticar, com perda de qualidade. Memoria sobrando nao muda isso.
- **Tempo**: contexto maior custa para ler o prompt e deixa cada palavra mais
  lenta, porque a placa rele o KV inteiro.
- **Qualidade**: modelo pequeno usa mal informacao espalhada num contexto longo.

## Como o harness ajuda o modelo local

- **Prefixo estavel**: ferramentas fixas por sessao e memoria na mensagem, para o
  Ollama reaproveitar a leitura da conversa. Medido: 2,7 s para 0,7 s num turno.
- **Resumo adiado**: o ponto de retomada so e escrito com a conversa parada, para
  nao ocupar a placa quando voce manda a proxima mensagem. Medido: turno seguinte
  de 9,2 s para 4,2 s.
- **Formato JSON forcado**: classificador e resumo pedem saida por JSON Schema,
  e o Ollama so deixa o modelo gerar o formato certo. Medido no qwen3:4b: resumo
  valido de 3 em 19 para 19 em 19, e classificacao certa de 3 em 16 para 14 em 16.
- **Janela respeitada pelo roteador**: conversa maior que a janela do modelo local
  vai para um modelo remoto.

## O que o modelo local nao faz bem

Explicar codigo lendo arquivos, com o qwen3:8b e 8k de contexto, nao foi
confiavel nas medicoes: sem verificacao, 8 de 11 respostas eram inventadas. Veja
[Cascata e verificacao](Cascata-e-verificacao).

## Custo local contra remoto

O DeepSeek fora do horario de pico ja custa fracoes de centavo por pergunta.
Rodar local economiza pouco dinheiro; o ganho real e privacidade, ausencia de
limite de taxa e funcionar sem internet.
