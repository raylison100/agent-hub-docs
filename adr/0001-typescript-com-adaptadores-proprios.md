# ADR 0001. TypeScript com adaptadores de provedor proprios

Estado: aceito. Data: 2026-09-09.

## Contexto

O produto precisa falar com Anthropic, DeepSeek, OpenAI e servidores locais
compativeis com OpenAI, e precisa ler o campo de uso de cada resposta com
precisao para o ledger. Duas familias de opcao foram avaliadas:

1. Biblioteca de abstracao pronta (Vercel AI SDK, LiteLLM, LangChain).
2. Interface propria `ProviderAdapter` com uma implementacao por formato de
   API, usando o SDK oficial de cada provedor por baixo.

Linguagem: o time ja trabalha com TypeScript no front (Vue) e o SDK oficial
de MCP e o Tauri tem integracao direta com Node.

## Decisao

TypeScript em todos os repositorios com codigo. Adaptadores proprios sobre
os SDKs oficiais `@anthropic-ai/sdk` e `openai`. Sem biblioteca de
abstracao intermediaria.

## Motivos

- Controle de custo exige os campos brutos de uso: `cache_read_input_tokens`
  na Anthropic, `prompt_cache_hit_tokens` no DeepSeek, `cached_tokens` na
  OpenAI. Bibliotecas de abstracao expoem isso de forma parcial e mudam o
  formato entre versoes.
- Parametros especificos importam para custo: `cache_control`, `effort` e
  fallbacks na Anthropic, `reasoning_effort` na OpenAI, troca de modelo no
  DeepSeek. Passar tudo por `providerOptions` de uma abstracao e o mesmo
  trabalho que escrever o adaptador, com uma camada a mais para depurar.
- Dois adaptadores cobrem todos os provedores previstos. O custo de manter
  a interface propria e baixo.
- Os SDKs oficiais ja tratam streaming, retry, tipos e erros. O adaptador e
  fino: traduz requisicao e resposta, nada mais.

## Consequencias

- Cada provedor novo com formato proprio exige um adaptador novo. Formatos
  compativeis com OpenAI entram apenas por configuracao de `base_url`.
- Testes de mapeamento de uso precisam de fixtures reais de cada provedor,
  atualizadas quando a API muda.
- O Claude Agent SDK nao e usado. Ele so fala com modelos Claude e o loop
  precisa ser um so para todos os provedores.
