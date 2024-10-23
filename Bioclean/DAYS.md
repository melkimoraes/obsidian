12/09
- ajustado dash PRODUTOS PENDENTES POR PERIODO
- feito os ajustes na parte de compras:
	- 1. Gerando de provisão do financeiro para pedido de compras, não está calculado os vencimentos na inserção, somente na alteração.
	1. Quando altera a data de entrega pelo portal de compras, não está preenchendo o campo AD_DTINICIO2, o campo só está sendo preenchido pelo Dash Follow-up.
	2. Ao duplicar o pedido de compra os campos de AD_DTINICIO2 e DTINICIO, não estão sendo zerados.


13/09
 - fizemos objeto novo na TGFITE - pra preencher a data AD_DTINICI2 corretamente!
 - criado botao de açao na TGFITE pra limpar data entrega, em caso de duplicar os pedidos!
 - também alterado objeto do alyson da Follow Up pra limpar Data Entrega!
 - campanhas de venda de combo, tudo que for PK tiramos do relatoriow
 - fizemos a VW_FATURAMENTOKIT  -> e CND DELA -> pra colocar no dash Faturamento Fiscal Detalhado que exibe tipo normal ou tipo kit -> pra trazer os kits!
 - Ajustamos erro em uma nota com erro por causa do tipo de titulo CARTEIRA


16/09
-  portal xml dando bó pra processar! -> STP TGFEST -> voltando ela ao normal , esse era o erro ao processar as notas.
- alterado relatorio formatado de Mapa de Separação e Mapa de Separação Crossdocking -> ambos alterado o canal, validado campos aonde tinha ligação com TGFVAR, e ajustado pra aparecer KIT e suas quantidades.
- vendo Relatorio formatados do pedido de venda. -> estava somando valos dos kits!
- Explicado como funciona as linhas vermelhas na Administração de Pedidos e as Etiquetas de Crossdocking, quando nao impresso todas as quantidades.
- Alterado o RecalculaFinanceiro e Recalcula Impostos pra comteplar rotina de recalculo dos valroes dos pedidos quando tiver kit!

17/09
- Feito um recalcular impostos java para Administração de Pedidos
- Feito um refazer financeiros java para Administração de Pedidos
- Feito a validaçao na dash da direita aonde as linhas de kit mostra o saldo do componente com menor estoque na Administração de Pedidos
- Vimos as validaçoes da cor amarela e vermelha na dash da esquerda na Administração de Pedidos, e fizemos alteraçao pra enxergar somente os produtos pendentes, entao em um faturamento parcial a linha fica de acordo com os produtos pendentes.
- Alterado um pedido aonde nao estava faturado e tava liberaçao adm, dando erro de estoque em um produto, pois ele contava como reservado daquele produto.

18/09
- Alterado Ação Agendada ATUALIZA DATAS FATURAMENTO - só data de faturamento! também filtrando ainda mais alguns grupos de TOP.
- Fizemos update em banco pras notas de venda que DTFATUR <> DHPROTOC por causa da alteração a cima e também alterado DTNEG <> AD_DTLIBADM.
- Feito a Dash PRÉ-NOTAS À FATURAR -> Apareça todas as pré-notas que foram administradas e  não foram faturadas na AD_LIBADM, como a data original do pedido(data de negociação) -> impressao mapa separação -> marca a data da impressao.
- Inserido campo da data da impressão do mapa de separação também!
- Ajustado as Regras de Negocio aonde trava alteração/inserção na CAB e ITE ignorando os KITS na TGFVAR.

19/09
- Erro ao fechar plp -> a busca das medidas estava incorreta, e também melhorei a busca dessas medidas, dando um erro mais claro se for o erro na embalagem.
- Ajustado etiquetas transportadoras -> bairro quando preenchido no endereço entrega , pegar de lá!
- Instalado em produção a melhoria da tabela de frete!
- feito trigger na TGFCPL e TGFPAR pra qualquer inserção/alteração -> se tiver CEPENTREGA no caso Endereço Entrega, valida se os outros campos estao preenchidos!

23/09
- Ajustado relatorio Diferença Valor Outros -> desconsiderando a duplicidade dos kits e também valor tava somando se produto na venda tivesse com mais de uma linha pra diferentes lotes!
- Ajustado Gerar MRP -> adicionado transação manual no java.

24/09
- flow cadastro de produtos
- ajuste na Adm de Pedidos -> na aba filha

27/09
- Feito o flow Solicitar Alteração Produto
- Alterado o flow Cadastro de Produto -> final do flow envia email confirmando cadastro do produto pro solicitando.

30/09
- Finalizado os ajustes no flow Cadastro de Produto, tendo agora tarefas novas como Qualidade, campos novos pras tarefas e codigo de barras colocado no final do processo.
- Iniciado ajustes no MRP -> alterando o Gerar Compra -> 


01/10 
- Feitos varios testes em relação ao Vlr. Outros juntamente com o Robson, colocando em eventos pra ser ajustado automaticamente.
- Continuando os ajustes no MRP -> Hoje alterado o Gerar MRP -> validando se solicitaçao e pedido pendentes, então excluindo. Também feito trigger se excluirem a solicitaçao direto na tela fazendo a msm exclusao.
	- Na inserção da solicitação alguns campos agora sendo inserindo na CAB com base no cadastro do Fornecedor e também preço do ultimo custo de reposição nos itens!
	- Encontramos o ajuste necessario pra ser dinamico as TOP's pra faturamento
- Ajudando também ao bater relatorio de vendas -> ajustamos a view faturamento e criei uma dash provisoria pra bater as notas com kit (Analise Vendas Kits).

02/10
- Ajuste novamente na rotina do Robson do valor outros, ajustamos a proc que estava com erro ao faturar.
- Alteramos Vlr unitario de 5 notas de venda ja aprovadas que eram antigas e tinham kit com vlr outros.
- Continuando os ajustes no MRP -> Hoje construido a nova dash de Solicitaçao de Compra com base na Follow up e também na antiga Solicitaçao de Compra -> só que agora olhando as solicitaçoes da TOP nova e também com botao de ação -> pra gerar o pedido de compra!


03/10
- Abertura - Pedidos -> colocado acesso de restrição de vendedor.
- Relatório Pedidos x Faturados -> Ajustamos o valor dos pedidos e temos o valor de pedidos geral em aberto.
- Diversos ajustes no Fechamento de Comissão ajustados juntamente com a Ju e o Erick. Visto erros por causa da alteração da data de baixa.


04/10
- Relatório Pedidos em Aberto -> -> colocado acesso de restrição de vendedor.
- Relatório Pedidos x Faturados -> colocar QTD dos pedidos por dia inseridos!
- Alteração flow Cadastro de Produtos -> últimos ajustes solicitados pelo comercial/marketing!
	- possível inclusão no flow de tabela de preços pro comercial cadastrar - pós inserção do produto!


10/10
- Parametro novo no relatorio de pedidos analitco e sintetico -> somente faturados.
- Filtro Padrao da tela de Ficha Cadastral atualizado -> vendo somente clientes da carteira dos vendedores se tiver vendedor no usuario, senao ve tudo.

11/10
- Criado filtro personalizado na Conciliação bancaria pra eles procurarem pelo Nro de Antecipação -> campo AD no Financeiro.
- Colocado no filtro padrao da Ficha Cadastral -> Se for Vendedor ve tudo tb no cadastro de Vendedores.

14/10
- Ajuste novamente na rotina do Robson do valor outros, ajustamos a proc que pra executar ela em algumas notas que o vlr outros estava correto na nota porem financeiros errados, fiz mais uma condição no final do objeto.
	- Por isso também proc, pra atualizar os que estavam errados.
- Erro no campo calculado de Antecipação na Conciliação Bancaria Ajustado colocando LISTAGG. Estava retornando mais de uma linha pro msm NUBCO nos Financeiros.

15/10
- Ajustamos o Gerar MRP -> trigger em partes fiz alguns ajustes pra parar o erro fantasma. Testei varias vezes com a Luciana.
- Ajustamos a Proc de Ajustar Vlr Outros novamente com o Robinho -> visto alguns casos que o valor outros ficou certo porem financeiros errados.
- Inicio do mapeamento da Plugg.to


21/10
Estoque atual → uma coluna com quanto que eu consumo de produto, com base em um range de data, tudo que a OP consumiu (aquela coluna na consulta que fica como Produção) → com base nisso.
quantidade que comprou → que entrou
media pelos dias de consumo

dias de estoque
consumo do produto 🆗
quantidade que comprou 🆗
média de consumo com base nos dias de estoque

TIRA COLUNA EMPRESA → VAI TER FILTRO DE EMPRESA, MAS ELE VAI SOMAR TUDO!!