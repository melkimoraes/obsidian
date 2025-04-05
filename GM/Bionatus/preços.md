calculo para formaçao de preço de venda.

simulação de preço
	regras de simulação de preços por estados

sao duas empresas -> bionatus e bionatus farma
	uma tela que vai usar pras duas empresas.
	bionatus laboratorio-> já tem um tela que o angelo de custo de produçao, só de custo nao de preço!

se a tela do padrao fizer simulaçao de custo com imposto por estado vai atender!

ver o video quando ele abrir o sistema antiga -> 22:15

formular a margem, do custo da fabricação pra dar o valor de venda.

nas produçoes em sankhya -> ele gera notas de produçoes e com isso da pra usar o valor do custo. que é a tabela do angelo que tem esses custos de produçao!

tela de simulação de preço -> o comercial vai usar!
vai subir as tabelas de frete deles -> tabela do rio

	o comercial só vai inserir uma porcentagem de comissao
	mas a tela tem que exibir pro comercial com base nos custos.

bionatus laboratorio -> usar o custo da tabela do angelo, que é o de produçao
bionatus farma -> preço de custo, tgfcus

bionatus -> 40 minutos que nao lancei horas de analise!


laboratorio-> pra clientes que querem vender marca propria!
	300 unidades de agua
	traz a mao de obra
	composiçao
	cliente é do RJ -> traz o valor da tributação
	origem/destino -> do estado! nao precisa por o cliente.

	 isso o vendedor pode ter comissao do representante/gerente, ele vai ter campo pra colocar a porcentagem livre.
	 e o valor do frete também!
	 valor de marketing

comercial -> informa o produto, e cada produto tem os processos que ele vai passar! qtd de cartucho, qtd materia prima. e dai coloca a qtd que ele quer vender, CAPSULAS/COMPRIMIDOS - isso divido por 30 da a quantidade do produto.

no excel eles tem o valor do custo da produçao -> com o valor desse custo entao ele começa

valor de mao de obra, materia prima, embalagem -> isso já é do custo mas tem que mostrar pro comercial!

despesa variaveis, comissao, valor frete,icms, pis/cofins, difal -> valor liquido da venda
comissao -> ele coloca o valor final de porcentagem
comissao e frete -> eles vao preencher!!
dv -> valor fixo por empresa.
% lucro -> por empresa, que o minimo é 12% fixo!
criar uma % lucro por produto


dai valor liquido % lucro -> sai o preco de venda


laboratorio -> licitaçao marca propria , empresa distribuidora empresa 4 
só pra distruibuidora eu acrescento o valor da transferencia pra la


---
Dani:

Laboratorio (Tela do Angelo para chegar ao calculo de custo de fabricação) ORIGEM DE TUDO.
Quando vender pelo laboratório, não terá custo de transferências, IMPOSTOS e FRETE, apenas Custo de fabricação + variaveis do simulador.
OU SEJA CUSTO DE FABRICAÇÃO = 5 + VARIAVEIS DE VENDA DO SIMULADOR
aqui é o codemp=1


VENDA INTERCOMPANY = LABORATORIO X DISTRIBUIDORA, O MESMO PREÇO DE CUSTO, SERÁ O PREÇO DE VENDA + IMPOSTOS(ICMS+PIS+COFINS+FRETE)
EXEMPLO: custo = 5 + ICMS = 1 + PIS = 1 + COFINS = 1 + FRETE = 1, PREÇO DE venda do laboratório para a distribuidora = 9 
aqui é tudo que for dirente do codemp <> 1 -> vai pegar o ultimo custo de reposição dos produtos daquela empresa!!

ESSE SERÁ O NOVO CUSTO DO PRODUTO ENTRANDO NA DISTRIBUIDORA = 9
OU SEJA, ASSIM QUE A NOTA DE VENDA DO LABORATORIO ENTRAR NA DISTRIBUIDORA, VOCÊ TERÁ O NOVO CUSTO DO PRODUTO PARA A DISTRIBUIDORA
O SIMULADOR TEM QUE PUXAR O ULTIMO CUSTO DE REPOSIÇÃO + AS VARIAVEIS DO SIMULADOR
9 + VARIAVEIS QUE PODERÁ REFLETIR EM UM VALOR FINAL MAIOR



solicitaçao moises :
SOLICITAÇÃO MOISES PARA O SIMULADOR:
O SIMULADOR DEVE PUXAR O ULTIMO CUSTO DE REPOSIÇÃO, MAS PRECISA DE UM CAMPO DO CUSTO AJUSTAVEL, PRA ELE VER O VALOR REAL SIMULADO E O VALOR COM O CAMPO DO CUSTO AJUSTAVEL,
OU SEJA QUANDO ELE PREENHCER O CAMPO DO CUSTO AJUSTAVEL, IRÁ DESCONSIDERAR O ULTIMO CUSTO DE REPOSIÇÃO.
o ultimo custo de reposição acredito que tenha que estar disponivel pra editar!

PIS/CONFINS É FEDERAL

O ICMS É RELATIVO POR ESTADO

TGFICMS -> TEM A ORIGEM E DESTINO -> VAI TER O ICMS, ICMS ST , DIFERENCIAL ALIQUOTA

JADER COMENTOU QUE TERIA QUE PEGAR O CADASTRO DO CLIENTE QUE TEM A CLASSIFICAÇAO ICMS

Tabela de PIS/COFINS = TGFIFE -> FILTRAR O CODEMP E PEGO A ALIQUOTA DELAS NA TOP 1101
Tabela de IPI = TGFIPI
Tabela de ICMS = TGFICM

TIPRESTRICAO -> SEM EXCECAO

 C - Consumidor Final Não Contribuinte -> É N NA TIPRESTRICAO
R - Revendedor -> PEGA OS QUE TEM GRUPO ICMS NO PRODUTO
X - Consumidor Final Contribuinte -> X NA TIPRESTRICAO

SEM RESTRICAO -> SEM RESTRICAO -> CODRESTRICAO = -1

ALÉM DA PORCENTAGEM ALIQUOTA SE TIVER REDUÇAO DA BASE USAR NO CALCULO.
PRODUTO 100 ->


CLASSIFICAÇÃO ICMS -> COLOCAR NO DOC -> REVENDEDOR
TGFPAR = CLASSIFICMS

SE O PRODUTO NAO TIVER PRO GRUPO ICMS DO PRODUTO

ICMS:
66,67 * 18% -> aqui os 18% é a aliquota de icms e os 66,67 é a reduçao se tiver vai entrar pro valor final do icms!

deu 66,67 por causa dessa formula:  ( 33,33 * 100 ) - 100 - 66,67

ICMS ST(se o produto tiver na aba Impostos/Informações por empresa o tipo de substituiçao : "Subst. na compra  e na venda"):
100 É O VALOR DO PRODUTO
 100*50%(mva) + 100 REAIS DNV = 150 * ALIQ SUBST. TRIBUTARIO = ICMS ST - ICMS = valor final

sempre vai ser o mva original nesse caso

empresa 2 e 1 que sao diferentes do simples nacional! farma -> toda venda pra outro estado, ele nao deve pegar o mva original, se for pra outro estado tem que ser o mva ajustado, como mas o mva ajustado?

quando a empresa for simples nacional -> vai usar o mva original

Cód. Regime Tribut.: campo na tsiemp -> se for difente do simples nacional!

mva ajustado -> planilha na pasta


- [x] ipi no custo -> alteravel porque as vezes o cliente nao paga IPI
- [x] frete no custo
- [x] setar nos campos de preço de venda algo fixo , talvez com parametros na empresa tb!: (confirmar no video!!)
	- [x] FRETE = 4%
	- [x] DESPESA FIXA = 14%
	- [x] COMISSAO = 10%
- [x] CRIAR UM CAMPO DE CUSTO ATUAL DE VENDA
- [x] fazer margem de lucro!
- [x] nas contas do lucro liquido! considerei o valor sugerido com desconto, blz??
- [x] CUSTO DO ANGELO NAO COLOQUEI AINDA PRA EMPRESA LA!f
- [x] 0807
- [x] faltou o erro do excluir!
- [x] contas:
	- [x] despesa fixa = porcDespesaFixa * precoVendaBruto
	- [x] lucro antes do IRPJ/CSLL = precoVendaLiquido - (despesa fixa + custoProdutoBruto)
	- [x] porc  lucro antes do IRPJ/CSLL =  lucro antes do IRPJ/CSLL/ precoVendaBruto
- [x] refazer todos os métodos que refazem calculo de porcentagem e valor pq tem que pegar as porcentagens ou valores da tela!
- [x] caso realmente os impostos de venda nao tenham nada haver com os do custo, tirar dos métodos da ext aonde recalcula porc/valor dos custos a chamada do PrecoVenda.
- [x] fazer as linhas de valor do preço de venda se alterar algum valor na mao e também as porcentagens do lucro antes/depois!
- [x] classificação icms na compra -> sempre revendedor talvez criar preferencia ou campo na empresa pra chumbar isso, no caso aquela classificaçao icms vai ficar pra venda!
- [x] e mudar a relaçao de estado de origem e destino! no caso o primeiro campo da onde to comprando do estado da empresa e o segundo pra onde eu vou vender sendo a origem estado da empresa também.
- [x] fazer os impostos de venda, usando destino e classICMS que ja ta puxando nos métodos principais!
- [x] TIRAR LUCRO DO PREÇO DE VENDA!
- [x] ver os impostos do preço de venda, pq eu nao posso somor todos os impostos lembrando que alguns somam!
- [x] colocar o campo Valor Liquido
- [x] colocar o campo Valor Sugeiro Sem Desconto
- [x] a conta do calculo do lucro liquido é do valor sem desconto (valor sugerido que vai virar valor liquido!)
- [x] alterar o desconto
- [x] colocar NCM na tela e vai salvar com ncm tb!
- [x] colocar o botao de ver impostos de venda do lado do impsotos na linha da venda
- [ ] colocamos o simular pelo NCM porém o calculo de icms nao esta para o NCM em sankhya e sim pelo grupo ICMS e tem diversos grupos ICMS pro msm NCM.


nao reduz -> se for simples nacional ou é contribuinte ou nao contribuinte


só na venda vou assumir a regra nova do jader do icms! se classificaçao ICMS for a opçao nova de SIMPLES NACIONAL ou CONSUMIDOR FINAL NAO CONTRIBUINTE E CONSUMIDOR FINAL CONTRIBUINTE.

essa regra só se for SP - SP PRA VENDA!!


se for simples nacional ou consumidor ou consumidor contruibuinte + e for venda e for SP - SP:

uma opção a +
REVENDOR SIMPLES NACIONAL