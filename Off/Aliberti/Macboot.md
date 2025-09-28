Tela de Pedido de Venda

Romaneio  → multiplos: 8, 10, 12, 15

Cabeçalho
Itens → aqui vai ser o produto generico

Gerar pedido de venda/nota de venda → aqui vai inserir os itens com base no EAN rateado pela numeração!

- [x] inserção dos itens vai ser conforme multiplos ou qtd normal?
- [x] pedido de venda ou venda vai inserir?
- [x] campo calculado statusnota da cab, nunota da cab
- [x] campo vinculado o nunota na TGFCAB → AD_NUNOTAPED
- [x] EVENTOS
- [x] PREFERENCIA! CODLOCALPEDVEND
- [x] ao atualizar a AD_TGFITE → atualiza o VLRTOT tb!
- [x] GET_LINK_TELA(‘br.com.sankhya.com.mov.CentralNotas‘, ‘Clique aqui para visualizar o Pedido‘, ‘{NUNOTA : 3}‘) - colocar clique aqui pra abrir o pedido?

---

Botão de ação na composição → vai criar outra composição do mesmo produto porem por numeração do sapato! 3,33% pra cada numeração. Só vai fazer esse calculo em cima da atividade de CORTE.
pra cima é 3,33% pra cima
pra menos é 3,33% pra menos

sempre vai aplicar da base que é o tamanho 39 → pro tamanho dele!
vai ter um campo lista do padrao (por ta trabalhando com controle de lista!) NA TPRLPA

só aplicar na TPRMPA

---

Relatórios:

Ficha de produção
- NUMPS na TPRIPRO

Agrupamento
- ficha é os números da OP

---

- Aparecer só as numerações do produto vou ter que mudar a tabela
	- quando inserir o produto ja inserir as numerações automaticamente zeradas
- Vendedor ajustado (estava com erro mesmo, ao alterar)
- Desconto: criar campo com percentual de desconto que o vendedor vai querer colocar e replicar isso nos itens do pedido.
- ireport do pedido(ta incluso?)
- Criar aba do tipo de negociação que o cliente vai poder usar.

---

Acessos

●
 Produção- macboot.sankhyacloud.com.br
●
 Teste - macboot-teste.sankhyacloud.com.br
●
 Aplicativos - app.macboot.sankhyacloud.com.br
●
 E-social - esocial-macboot.sankhyacloud.com.br/index.html



 Senha do usuário SUP :


●
 Produção - 0VG42OPm4qoDGf3mp6
●
 Teste - 0VG42OPm4qoDGf3mp6
 Senha do WPM
●
 Produção - 6f7bZ7ZowEno
●
 Teste - 6f7bZ7ZowEno