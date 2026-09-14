# Seguranca

## Onde ficam os segredos

- **Chaves de API**: cifradas no SQLite do daemon, cadastradas pela tela Chaves.
  Nenhum cliente recebe a chave de volta.
- **Credenciais de conectores**: o `mcp.json` so guarda `${VARIAVEL}`; o valor
  fica no cofre.
- **Configuracao da maquina**: `mcp.json` e `config.toml` nao vao para o git.
- **Saida das ferramentas**: passa por redacao de padroes de segredo
  (`policies/secrets.json`) antes de chegar ao modelo e aos clientes.

## O que o agente pode fazer

- Ferramentas classificadas por risco: `read`, `write`, `exec`.
- A politica do perfil ou do papel diz `allow`, `ask` ou `deny` para cada risco.
- Comando destrutivo (`rm -rf /`, `git push --force`, `git reset --hard`,
  `git clean -f`, `DROP TABLE`, `mkfs`, `dd`, `shutdown` e afins) pede aprovacao
  em qualquer modo, inclusive no automatico.
- Caminhos fora do workspace sao recusados.
- `git` so roda em pasta que e repositorio de verdade.
- Chamada de ferramenta e validada com JSON Schema antes de executar.
- `sandbox` no perfil roda comandos dentro de container, sem rede se quiser;
  execucao liberada sem sandbox e rebaixada para pedir aprovacao.
- Aprovacao pendente expira.

**Modelo local nao e modelo confiavel**: as mesmas regras valem para ele.

## Quem pode falar com o daemon

- Escuta so em `127.0.0.1` por padrao.
- Na propria maquina, a credencial e entregue por `/pair/local`, que exige
  loopback e origem da propria interface. Ressalva: qualquer processo local pode
  pedir essa credencial; o limite de confianca e o seu usuario na maquina.
- De fora: senha com scrypt, credencial por dispositivo guardada como hash,
  revogacao individual e espera crescente depois de erros.
- Pelo relay: cifra ponta a ponta; o relay nao le o conteudo.
- Webhooks de entrada: assinatura conferida por fonte, filtro e deduplicacao.

## Conteudo de terceiros

Corpo de webhook entra como dado delimitado na mensagem, nunca como instrucao de
sistema. Resultado de ferramenta volta ao modelo como resultado de ferramenta,
com segredo redigido e cortado no teto de tamanho.

## Compartilhamento de modelos

Quem compartilha ve o texto das mensagens que chegam ao modelo dele. So atende
chamada de modelo, dos modelos liberados, dentro do limite diario, e cada convite
tem sala e chave proprias. Do lado de quem usa, o token da sala fica cifrado e
fora do ambiente do processo. Detalhes em [Compartilhar modelos](Compartilhar-modelos).

## Automacao

Toda automacao exige orcamento, comeca em rascunho e respeita o interruptor
geral.

## Relatar um problema

Encontrou uma falha de seguranca? Abra uma issue sem detalhes de exploracao
pedindo contato, ou fale com o autor pelo GitHub.
