# Conectores MCP

Conectores sao servidores MCP que dao ferramentas novas aos agentes: GitHub,
GitLab, Jira, bancos, sistemas de pagamento, o que existir.

## O mcp.json nao vai para o git

Os conectores ficam em `mcp.json` na sua pasta de agentes. Esse arquivo lista os
servidores que voce usa e caminhos da sua maquina: e configuracao local, esta no
`.gitignore` e nao deve ir para nenhum repositorio. O projeto traz um
`mcp.example.json` para comecar.

## Cadastrar

Pela tela Configuracoes, **Conectores**:

- **Importar do Claude Code**: le os servidores que voce ja tem configurados la.
- **Colar JSON**: no formato `mcpServers` do Claude Code ou no formato do projeto.
- Qualquer valor que pareca credencial vira referencia `${VARIAVEL}` e o valor
  vai para o cofre cifrado. O `mcp.json` nunca guarda segredo.

Formato:

```json
{
  "servers": {
    "github": {
      "command": "python3",
      "args": ["/caminho/do/servidor/server.py"],
      "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" },
      "risk": {
        "github_list_*": "read",
        "github_get_*": "read",
        "*": "write"
      },
      "enabled": true
    }
  }
}
```

Servidor HTTP usa `url` e `headers` no lugar de `command` e `args`.

## Risco por ferramenta

O mapa `risk` classifica cada ferramenta por padrao de nome: `read`, `write` ou
`exec`. A politica do agente decide o que acontece com cada risco. O que nao
casar com nenhum padrao cai em `*`. Use o nome completo como o servidor expoe
(`github_list_commits`, nao `list_commits`).

## Ligar em um agente

No perfil, `tools.mcp: [github]`. As ferramentas entram com o prefixo do servidor
(`github__github_list_commits`).

## Conectar e desconectar

Na tela, cada conector tem **Conectar**, **Desconectar** e **Remover**. Conector
desconectado por voce nao e ligado sozinho nem gera aviso nos runs. Conector que
cai sozinho e religado na proxima verificacao.

## Muitas ferramentas, janela pequena

Quando o catalogo nao cabe em 25% da janela do modelo, entram as ferramentas mais
relevantes para o pedido, com um minimo de tres. O conjunto fica fixo na conversa
e so e refeito quando voce cita um servidor ou ferramenta que ficou de fora.

## OAuth

Servidor HTTP que exige OAuth ganha um bloco `oauth` com `issuer` e `scopes`. O
daemon descobre os endpoints, registra o cliente quando o provedor permite, e
voce clica em **Autorizar**. O token vai para o cofre e renova sozinho.
