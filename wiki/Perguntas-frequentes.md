# Perguntas frequentes

**Preciso de chave de API?**
De pelo menos um provedor, ou de um modelo local no Ollama. Com mais provedores o
roteador tem mais opcoes para economizar.

**Minhas chaves vao para algum servidor?**
Nao. Ficam cifradas no daemon, na sua maquina, e so saem na chamada ao proprio
provedor.

**Qual modelo ele usa?**
Depende de cada mensagem. A conversa mostra qual agente respondeu e por que. Voce
pode fixar um agente se preferir.

**Como sei quanto gastei?**
Cada resposta mostra o custo, e Configuracoes, Custos tem o relatorio completo com
exportacao CSV.

**Mudei um perfil e nada aconteceu.**
Use Recarregar configuracao em Configuracoes, Conexao, ou `make reiniciar`.

**Fechei o app e o daemon continuou rodando.**
E o esperado. O daemon e independente dos clientes. Com `make servico` ele sobe
com a maquina e volta sozinho se cair.

**Posso usar no trabalho, na empresa?**
Nao sem autorizacao. A licenca proibe uso comercial. Veja [Licenca](Licenca).

**Conector desconectado aparece como erro em todo run?**
Nao. Conector que voce desligou e ignorado em silencio; so o que caiu sozinho
gera aviso.

**Da para usar do celular?**
Sim, pela interface instalada como PWA, com senha ou pelo relay. Veja
[Desktop, celular e acesso remoto](Desktop-celular-e-acesso-remoto).

**O modelo local substitui os pagos?**
Para classificar, resumir e reescrever pedido, sim. Para explicar codigo lendo
arquivos, com modelo pequeno, as medicoes mostraram que nao. Veja
[Modelo local](Modelo-local).

**Por que tantos repositorios?**
Cada parte tem ciclo e dependencias proprios: a interface pode mudar sem
recompilar o daemon, o relay roda num servidor, os agentes sao so texto. Para
usar, nao precisa de nenhum deles: o pacote unico junta o necessario, veja
[Instalacao](Instalacao).
