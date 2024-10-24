
https://developers.plugg.to/reference/getting-started-with-your-api

shopee
mercado livre
mercado livre full -> só o full emite nota
b2w
cnova
magalu
olist
drogaraia
beleza na web

ação agendada -> na plugg.to vendo se tem pedido pra integrar em sankhya!

oq o sankhya vai ter que integrar na plugg.to -> preço e estoque?

preço
	-> no antigo sistema era usado a tabela de preço VTEX e eles colocavam uma porcentagem maior em cima pra vender na plugg.to
	preço normal tanto os marktplace como o full!

estoque
		-> tb atualiza tudo na plugg.to -> tem o local proprio no sankhya ja E-COMMERCE.
		-> muda pro mercado full ! pq ele vai pra estoque em poder de terceiro! 
	O ESTOQUE DOS MARTPLACE a gente atualiza
	O FULL ele atualiza em sankhya!

frete
	-> a plugg.to emite as etiquetas pras transportadoras devidas do marketplace.


tudo na empresa 4 somente.

Ações Agendadas:
	- Busca de pedidos na plugg.to
		- Se for full -> nao é pedido e sim nota!
	- Atualiza Preço dos produtos na plugg.to! 
	- Atualiza Estoque dos produtos somente nos MKTPL da plgg.to
		- Se for full -> atualiza o estoque no sankhya, do estoque de terceiros.

Tabelas:
	- Depara de MTKPLACE
	- Depara de Produtos/MTKPLACE?
	- Depara do Frete


plugtoo -> ele quer usar cotação de frete, gerencia preço lá, varias coisas que na vtex nao tem!

---

Mapeamento Principal
	- Produto
		- vai querer selecionar o produto do sankhya e enviar pra plugg.to -> e dai quando inserir retornar o id plug.too em sankhya também.
	- Marktplace -> o marktplace -> vai ter que importar 
		- e variso campos que vamos usar pra tabela de preço, tipo de negociação, empresa, vendedor.
	- Preço
		- o preço vai ser por marktplace, entao no sankhya vmaos ter uma tabela pai PLUGGTO e dai as filhas por marktplace!
		- Ação agendada pra atualizar o preço de tempo em tempo.
		- ter um forçar preço
	- Estoque
		- Ação agendada pra atualizar isso de tempo em tempo.
		- estoque da 199 pra todos os marktplace.
		- pra kits -> pegar o menor estoque componente do kit, esse é o estoque do kit.
	- Pedido
		- Forçar o pedido, e um numero de pedido.
	- Fullfilment
		- Vai controlar o estoque de terceiro desse cara. Vai pegar a nota de remessa que eles geram no mercado livre, e dai essa nota em sankhya vai tirar do estoque e vai pra estoque de terceiro.
		- controlaar as notas de devoluçoes do mercado full -> pra voltar no estoque
	- Etiqueta
		- quando forçado o pedido a gente capturar a etiqueta pra eles imprirem em sankhya.
	- Tipo de negociação -> se for padrao da plugto sem ser por marktplace -> vou fazer uma tela pra fazer depara.

- [ ] Etiqueta -> eles enviam ela emitida e enviam a etiqueta impressa?
- [x] Como anexar o produto aos marketplaces -> é pela tabela de preço?
- [ ] Cadastro de produto já pede um preço e preço especial fora do cadastro do produto??
- [ ] Oq diz pra plugg.to que quero parar a venda de um produto?? Tipo desativar!?
- [ ] Imagens → precisa ser url → ver com o erick

---


Exceção do produto → 
	DIMENSOES EMBALAGEM → MANDAR VAZIO ok
	Origem do Produto → Procedência Nacional → criar campo na tela de produto plugg.to
	Criar todos os campos que estamos subindo como cadastro do produto! os que puxamos do cadastro do produto sankhya, deixa somente leitura  e o de origem como é um campo especifico plugg.to deixa pra preencher obrigatorio!
	Imagens → TESTAR AS ABAS DE ANEXOS DO CADASTRO DO PRODUTO

nas ações agendadas de atualização de preço e estoque assim como no processar do produto → só substitui o log do estoque/preço caso tenha alterado esse valor!