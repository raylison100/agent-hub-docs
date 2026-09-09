# ADR 0002. Tauri como casca desktop, daemon Node como sidecar

Estado: aceito. Data: 2026-09-09.

## Contexto

O desktop precisa rodar em Windows, macOS e Linux, embutir a interface web e
manter um processo de agente vivo. Opcoes: Electron, Tauri, aplicacao
nativa por plataforma.

## Decisao

Tauri 2 para a janela, bandeja e instaladores. O daemon Node roda como
sidecar iniciado pelo Tauri, ou como servico do sistema quando o usuario
preferir. A interface e o mesmo build Vue servido nos outros clientes.

## Motivos

- Binario pequeno e uso de memoria menor que Electron, o que importa em um
  aplicativo que fica aberto o dia todo na bandeja.
- A logica de agente e Node de qualquer forma (SDKs, MCP). Separar em
  sidecar mantem o Rust minimo e permite que o daemon exista sem o desktop.
- Instaladores para os tres sistemas saem do mesmo pipeline.

## Consequencias

- Empacotar o daemon como executavel unico por plataforma (`node
  --experimental-sea` ou equivalente) para o sidecar. Isso entra na fase 2.
- O webview de cada sistema tem diferencas. A interface deve ser testada em
  WebView2, WebKit e WebKitGTK.
- Se o sidecar falhar em alguma plataforma, o desktop conecta a um daemon
  instalado como servico. O produto nao depende do sidecar funcionar em
  todo lugar.
