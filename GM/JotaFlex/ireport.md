quantidade de clientes -> 1:09

- [x] tem 3 relatorios que tem 2 pendencias especificas aqui embaixo ja marquei
- [x] Orçamentos por cliente origem Detalhe [Report] -> NOVO -> esperando robinho criar o campo na TGFPAR pra colocar na condicional do inner join !! -> ele fez só altgerar!
- [x] e o dash FATURAMENTO ANUAIS
- [x] colocar nos relatorios o filtro de vendedor logado:
- [x] -> colocar periodo de data que nao teve orçamento ow nf , pra ficar de facil compreensao
- [x] -> verificar o valor da perda se ta ok e ver somatoria das porcentagens do perc. perdas
- [x] ver Orçamento por Cliente Origem 
- [x] ver Clientes Novos


Grupo item [report] -> 🆗
		- Trazer dados igual ao de Faturamento Geral [Report] 
		- Mesmo sem valor trazer o produto com 0
		- Filtros
			- Vendedor 
				- Dentro de vendedor buscar os dados do subcadastro de grupos de produtos para saber quais devem ser exibidos.
		 - colocar no cabeçalho, grupo item o nome do grupo item! nao o codigo, na regiao colocar vendedor e apelido
		 - nao filtrei nada de grupo de produto

Faturamento Geral [Report] -> 🆗
		- Filtros
			- Vendedor (região)
				- Campo de texto para informar varios vendedores.
			- Ano Vigente
			- Ano Comparativo - se null trazer o ano anterior ao vigente
			- Grupo de Produtos
	-> nao é bem isso aqui de filtro! é o IREPORT VENDA-POR-VENDEDOR

Relatório de notas fiscais [Report] -> OK -> 🆗
		- Agrupamento é por vendedor.
		- Verificar parte de comissão com o Bruno.
-> Ireport -> VENDAS DE CLIENTE/VENDEDOR (RESUMO)
		-> parte debaixo com Devoluçoes do vendedor \periodo:
		-> tirar coluna de NF, e agrupar por cliente, pode coloar codigo do cliente.
		-> agrupamento no final de todo o faturamento somado os vendedores.
		-> ver a grafia do relatorio todo!
		-> mudar nome pagina para pt

Quantidade de clientes [Report] -> OK -> 🆗
		- Coluna "Região" trocar para VENDEDOR
		- Resultado é total de positivação de clientes dentro do mes (emissao de nota)
			- Vendas ou serviço.
		- Filtros
			- Vendedor 
				- Dentro de vendedor buscar os dados do subcadastro de grupos de produtos para saber quais devem ser exibidos.		
			- Ano Vigente
			- Ano Comparativo - se null trazer o ano anterior ao vigente
			- Grupo de Produtos -> ser multi parametro igual do vendedor, mas imagina filtrar todos os codigos de produto !?
			 -> Aumentar coluna Vendedor
	
Orçamentos por cliente origem [Report] -> OK -> 🆗
		- Neste deve considerar o agrupamento da tabela AD_ORIGCONTATO 
		- Traz o resumo de orçamentos com o seu total, caso ele ja tenha sido faturado trazer o valor na coluna OV. (ligar com  TGFVAR)
		- > trocar Vlr PV, Qtd. PV's
		-> QTD ORÇAMENTOS , QTD PV
		 -> vlr total orçado  - vlr total vendido -> VER SE OS VALROES CONVERSOES ESTAO OK
		 -> TOTAL NO FINAL DO RELATORIO POR TODAS AS ORIGENS 
		 -> Ajustar tamanho das colunas e aumentar a coluna do cliente e valores!
		 -> desconsiderar clientes sem origem!
		 -> ajustar o nome dos parametros lá em cima preenchidos! ta colocando o codigo e nao o nome

Orçamentos por cliente origem Detalhe [Report] -> NOVO -> esperando robinho criar o campo na TGFPAR pra colocar na condicional do inner join !!
		- Neste deve considerar o agrupamento da tabela AD_ORIGCONTATODET
		- Traz o resumo de orçamentos com o seu total, caso ele ja tenha sido faturado trazer o valor na coluna OV. (ligar com  TGFVAR)	
		-> diminuir titulo do subgrupo dos detalhes -> sem ser BOLD TB
		-> desconsiderar clientes sem origem!
		-> colocar titulo do total geral da origem
		 -> TOTAL NO FINAL DO RELATORIO POR TODAS AS ORIGENS 

Clientes inativos sem orçamento [Report] -> 🆗
		- Precisa cadastrar
		- 
		- Filtros
			- Informar periodo.
			
Clientes inativos Sem NF [Report] fiiz o -> CLIENTES INATIVOS por enquanto só se tratando de NF! -> 🆗
		- Precisa cadastrar
		- 
		- Filtros
			- Informar periodo.
			- acrescentar uma data que seria apartir dessa data pra frente que vou trazer os inativos
		- trocar regiao por vendedor ! trocar todas regiao -> vendedor!
		- codigo "-" cliente
		- total de clientes (quantidade) e total de valor! SOMATORIO LA EMBAIXO

Conversão de Vendas [Report] -> OK -> 🆗
		- Validar vendedor para impressão. (regra para que cada vendedor veja apenas suas vendas) ok
		- Tratar campos nulos. ok
		- Agrupar resultado dos motivos ok
		- Tratar data para trazer orçamentos ou pedidos de venda convertidos dentro de determinado periodo.  -> ver se o pedido de venda ta buscando do periodo e nao necessariamente vinculado ao orçamento do periodo!
		- item (8) - QTD VENDIDO. adicionar PERC. QTD VENDIDO. -> PERC. VENDIDO NAO É ISSO!?
		- Data e pagina em portugues, usuario de impressão. -> OK
		-> ver video desse realtorio pediu alteraçao dos labels!
		 ->

Relatorio -> ver relatorio que ela pediu de clientes origem no video! relacionado a grupo de produto.



---

Relatorio:
	Conversão de Vendas [Report]
		- Validar vendedor para impressão. (regra para que cada vendedor veja apenas suas vendas)
		- Tratar campos nulos.
		- Agrupar resultado dos motivos
		- Tratar data para trazer orçamentos ou pedidos de venda convertidos dentro de determinado periodo. 
		- item (8) - QTD VENDIDO. adicionar PERC. QTD VENDIDO.
		- Data e pagina em portugues, usuario de impressão.
		
	Clientes novos [Report]
		- Validar vendedor para impressão.(regra para que cada vendedor veja apenas suas vendas)
		- Coluna Notas ativas - trocar para NF Ativ. e preencher com o nro da primeira nota emitida para o cliente.
		- Sub report traz quantidade de novos clientes e valor faturado para eles.
		- Imprimir os filtros
		- Filtros
			- Vendedor (região)
				- Campo de texto para informar varios vendedores.
			- Data
			- Grupo de Produtos.
		- Data e pagina em portugues, usuario de impressão.
		
	Conversão mensal [Report]
		- Validar vendedor para impressão.(regra para que cada vendedor veja apenas suas vendas)
		- Não esta agrupando o mês
		- Mesmo sem volume de informação em determinado mês, devera apresentar o mês com valores zerados.
		- Filtros
			- Vendedor (região)
				- Campo de texto para informar varios vendedores.
			- Ano Vigente
			- Ano Comparativo - se null trazer o ano anterior ao vigente	
			
	Devoluções [Report]
		- Coluna "Região" trocar para VENDEDOR
		- Filtros
			- Vendedor (região)
				- Campo de texto para informar varios vendedores.
			- Ano Vigente
			- Ano Comparativo - se null trazer o ano anterior ao vigente
			- Grupo de Produtos

	Faturamento [Report]
		- Coluna "Região" trocar para VENDEDOR
		- Filtros
			- Vendedor (região)
				- Campo de texto para informar varios vendedores.
			- Ano Vigente
			- Ano Comparativo - se null trazer o ano anterior ao vigente
			- Grupo de Produtos
		
	Faturamento Geral [Report]
		- Filtros
			- Vendedor (região)
				- Campo de texto para informar varios vendedores.
			- Ano Vigente
			- Ano Comparativo - se null trazer o ano anterior ao vigente
			- Grupo de Produtos
	
	Grupo item [report]
		- Trazer dados igual ao de Faturamento Geral [Report] 
		- Mesmo sem valor trazer o produto com 0
		- Filtros
			- Vendedor 
				- Dentro de vendedor buscar os dados do subcadastro de grupos de produtos para saber quais devem ser exibidos.

	
	Quantidade de clientes [Report]
		- Coluna "Região" trocar para VENDEDOR
		- Resultado é total de positivação de clientes dentro do mes (emissao de nota)
			- Vendas ou serviço.
		- Filtros
			- Vendedor 
				- Dentro de vendedor buscar os dados do subcadastro de grupos de produtos para saber quais devem ser exibidos.		
			- Ano Vigente
			- Ano Comparativo - se null trazer o ano anterior ao vigente
			- Grupo de Produtos
			
	Orçamentos por cliente origem [Report]
		- Neste deve considerar o agrupamento da tabela AD_ORIGCONTATO 
		- Traz o resumo de orçamentos com o seu total, caso ele ja tenha sido faturado trazer o valor na coluna OV. (ligar com  TGFVAR)
		
	Orçamentos por cliente origem Detalhe [Report]
		- Neste deve considerar o agrupamento da tabela AD_ORIGCONTATODET
		- Traz o resumo de orçamentos com o seu total, caso ele ja tenha sido faturado trazer o valor na coluna OV. (ligar com  TGFVAR)	
		
	Analise de Vendas [Report]
		- Região trocar pelo inicial do nome do vendedor
		- Verificar agrupamento, muitas linhas duplicadas.
		- Filtros
			- Clientes
			- Grupo
			- Vendedor 
			- Periodo
			
	Relatório de notas fiscais [Report]
		- Agrupamento é por vendedor.
		- Verificar parte de comissão com o Bruno.
		
	Clientes inativos sem orçamento [Report]
		- Precisa cadastrar
		- 
		- Filtros
			- Informar periodo.
			
	Clientes inativos Sem NF [Report]
		- Precisa cadastrar
		- 
		- Filtros
			- Informar periodo.