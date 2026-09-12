# ADR 0005: nao adotar a compactacao server-side da Anthropic como padrao

Data: 2026-09-12
Situacao: decidido

## Contexto

A Anthropic oferece compactacao no servidor pelo bloco `context_management.edits`
com `type: compact_20260112` e o cabecalho beta `compact-2026-01-12`. Quando a
entrada passa do gatilho (padrao 150 mil tokens, minimo 50 mil), a propria API
resume a conversa, devolve um bloco `compaction` no conteudo e passa a descartar
tudo que vem antes dele. Funciona com cache de prompt, streaming e ferramentas, e
o uso vem detalhado em `usage.iterations`.

Hoje o Agent Hub compacta do lado de ca: `compactHistory` poda resultados de
ferramenta e, quando precisa, manda a conversa para o modelo declarado em
`context.summarizer` do perfil, que e o qwen3 local.

## Decisao

Continuar com a compactacao propria. Nao ligar a compactacao server-side por
padrao.

## Motivos

1. Custo, que e o proposito deste projeto. A iteracao de compactacao e cobrada
   por inteiro: resumir 180 mil tokens no Opus 5 custa cerca de 0,90 USD de
   entrada mais a saida, quase 1 USD por compactacao. O mesmo resumo no qwen3
   local custa zero, e no gemini, que tem janela de 200 mil, custa cerca de
   0,14 USD.
2. Vale so para um provedor. Metade dos runs aqui roda em DeepSeek, Gemini,
   OpenAI ou local; uma solucao que so serve para a Anthropic deixaria dois
   comportamentos diferentes de contexto convivendo no mesmo harness.
3. O controle sai da nossa mao. Hoje decidimos quando compactar (`compact_at` por
   perfil), o que podar primeiro e quem resume. La o gatilho e so por tokens de
   entrada e o minimo e 50 mil.
4. O ganho real que ela traz, preservar melhor o raciocinio e evitar uma ida e
   volta, nao paga a diferenca de preco no nosso caso.

## O que a avaliacao revelou de defeito nosso

O sumarizador fixo no qwen3 quebra quando a conversa passa da janela dele, de
8192 tokens: justamente o caso em que a compactacao e necessaria. Corrigido no
mesmo dia: `summarizerThatFits` usa o modelo do perfil enquanto a transcricao
couber em 70% da janela dele e, acima disso, cai para o modelo remoto declarado
em `prompt_improver.remote_agent`, que tem janela grande e continua barato.

## Quando reabrir

- Se a Anthropic passar a cobrar a compactacao a preco de cache ou de lote.
- Se aparecer um agente que so roda na Anthropic e vive perto do limite de
  janela, ai vale ligar por perfil, em `provider_options`, sem virar padrao.
