- [x] Pedir chavenfe que foi transportado pela Ativa e uma numero de nota fiscal que foi transportado pela total express!
- [x] ajustar as apis da DLOG na bioclean pro codemp correto!
- [x] tracking da ATIVA PARA FAZER CHAVE: 35240212362748000113550010001575741930782156
- [x] MUDAR TAMANHO DO CAMPO STATUS  DO  GMITETQTKG
- [x] ver se a trava da cab ta on na bioclean! 🆗TRG_U_TGFCAB_GMIT
- [x] CAMPO CALCULADO DO NUMERO DA ETIQUETA
- [x] CAMPO CALCULADO COM URL DO LINK DE RASTREIO
- [x] colocar url rastreio endereço site -> nas config das transportadoras - CORREIOS E JADLOG!
- [x] fazer ação agendada pro tracking!?
	- [x] campo ENTREGUE GMITETQ,GMSIGPLP
	- [x] campo STATUSENTREGUE GMITCFG
	- [x] preferencia  GMITDIASCSLTETQ
	- [x] campo STATUS MUDAR PRA VARCHAR2(200) nas duas tabela de tracking
- [x] excluir tabela do erick, excluir preferencia e ver o doc!
- [x] ação agendada na base da bioclean do rastrear!
- [x] ver ação agendada do SIGEP na ext! -> TESTAR LOCAL? com algum idplp do bruno lá talvez!?
- [x] ver o role do sedex / pac na rotina do bruno como ele ta lendo isso! falo no java da ext !!! pra ver se ta certo, ao gerar etiqueta no SIGEP
- [x] ver no gerar etioqueta se eu to validando se inseriu embalagem ativa ou só das transportadoras via API! pq consegui gerar etiqueta da ativa!
- [x] cotar frete -> diferente -> pega transportadora, frete , mas só vai inserir embalagens
	- [x] vai precisar de um campo do serviço provavelmente
	- [x] aqui tenho minhas duvidas se nao vai gerar qtd de embalagens diferentes da que foi informada!
- [x] quando começar a testar as etiquetas, correios principalmente ver se ta indo ok SEDEX/PAC!
- [ ] fazer dash cotaçao de frete!
- [x] colocar no log da integraçao transportadoras qualquer vtex integraçao com erro!
	- [x] -> COLOCAR NOS CONTINUE UM TTHROW E NO TRY CATCH DAI JA NAO VERIFICAR SE JA NAO LOGOU AQUELE MSM ERRO!! -> TESTAR
- [x] preencher serviço  cab-> ao selecionar frete!?
- [x] colocar serviço no pedido de vnda também
- [x] ver correios pq as etiquetas nao tao conseguindo ser bipadas.
- [x] dlog ta inserindo etiqueta automaticamente -> nao consegue gerar etiqueta em sankhya
	- [x] aqui talvez deixar imprimir etiqueta e fodase a tela de etiquteasa?? nao vai rastrear DLOG??
	- [x] bypass quando retornar esse erro de que ja existe!
- [x] e ver nota com erro correio -> com frete cotado em outra transportadora
	- [x] -> aqui talvez colocar nas validaoes de gerar etiqueta -> transportadora selecionada diferente da nota!!!!!
	- [x] trava ao confirmar pedido de venda se a transportadora tiver diferente da cotada!? caso tenha cotaçao de frete pro pedido!
- [x] VER ERRO DOS CORREIOS -> TENTAR NA NOTA DAI 26045

SELECT NUNOTA,(SELECT NOMEPARC FROM TGFPAR WHERE CODPARC=CODPARCTRANSP),STATUSNFE FROM TGFCAB WHERE (NUNOTA IN (
       SELECT NUNOTA FROM TGFVAR WHERE NUNOTAORIG IN (
SELECT NUNOTA FROM GMITFRE WHERE SELECIONADO = 'S' )) OR NUNOTA IN (
SELECT DISTINCT NUNOTA FROM TGFVAR WHERE NUNOTAORIG IN (
SELECT DISTINCT NUNOTA FROM TGFVAR WHERE NUNOTAORIG IN (
SELECT NUNOTA FROM GMITFRE WHERE SELECIONADO = 'S' ))
)) AND TIPMOV='V' AND STATUSNFE='A'


# Pontos melhoria cotaçao - erick
- [x] transportadora só atende essa regiao! -> dizer uma regiao que a transportadora atende
	- [x] criar uma liberação pra questao da regiao , por exemplo o melhor frete foi uma transportadora diferente da regiao, gera essa liberação.
- [x] frete gratis
	- [x] televendas -> se o cliente é dentro de sp, e comprar acima de 300 o frete é gratis! fora do estado de SP e for 500 o frete é gratis
	- [x] ou campnhas acima de 200 reais é frete gratis.
	- [x] fixar por enquanto televendas -> incluso e varejo -> extra nota.
- [x] frete cobrado -> pode ter tido fretee gratis por alguma regra , mas ele quer o frete gerencial! msm com frete gratis. -> quando cotar frete -> eles selecionam extra nota ou incluso! 
- [x] ver com o erick alteraçoes no gerar etiqueta aonde for possivel gerar etiqueta na nota de venda sem estar confirmada alterar -> dai entao fazer o documento de gerar etiqueta e rastreio.
	- [x] jadlog -> obriga nf emitida porque o tipo do documento fiscal está sendo o de NFE , mas tem a opção CTE, vai ser uma opçao!?
	- [x] Dlog -> precisa!
	- [x] correios -> precisa!
- [x] frete motoboy -> 15 reais -> criar marcação MOTOBOY NO CODPARC, esse marcação pula nas trigger de exige cotaçao de frete!
- [x] ver se vai entrar o outros transportadoras!




ele compra o fardo da caixa das embalagens. vem 150 caixas desmontadas


24010076810 - ENTREGA EXPRESSA

37,94 - PESOBRUTO

5,42055


37,94388 * 0,01310400 = 0,49721660352 / 0,08543566 = 5,819778339864174

0,35

37,94 - 0,35 = 37,59
0,08543566 - 0,01310400 = 0,08462803

TOTAL EXPRESS TESTE -> 




---

SELECT CODPROD 
FROM (
	SELECT CODPROD, MIN(GMM3) AS MIN_M3  
	FROM TGFPRO  
	WHERE CODGRUPOPROD IN (SELECT CODGRUPOPROD FROM TGFGRU WHERE GMEMBALAGEMIT = 'S')     
	AND ATIVO = 'S' 
	AND NVL(GMM3, 0) > 0.000001     
	AND M3 >= 0.08543566  GROUP BY CODPROD  ORDER BY MIN_M3
	)
WHERE ROWNUM = 1


SELECT CODPROD 
FROM (
	SELECT CODPROD, MAX(GMM3) AS MAX_M3  
	FROM TGFPRO  
	WHERE CODGRUPOPROD IN (SELECT CODGRUPOPROD FROM TGFGRU WHERE GMEMBALAGEMIT = 'S')     
	AND ATIVO = 'S' 
	AND NVL(GMM3, 0) > 0.000001   
	GROUP BY CODPROD  
	ORDER BY MAX_M3 DESC)
WHERE ROWNUM = 1

---
responde dlog:
{"response":{"codStatus":1,"descrStatus":"Success: Cliente Habilitado para utilizar API","insertNf":[{"nfChave":"35240539349016000140550020000696321971454653","codSubStatus":1,"descrSubStatus":"NF inserida com sucesso","linkRastreamento":"https://dlog.com.br/rastreamento/rastreio.php?codId=39349016000140-05668374556-69632-9636322"}]}}


----

- [x] ver a questao da homologaçao!
- [x] criar tipo taxa emex que tem tudo na msm linha
- [x] TERMACO
	- [x] GRIS -> PELO VALOR DA NOTA 
	- [x] PEDAGIO -> NAO É FIXO NÉ? pego o peso / 100 * valor do pedagio
	- [x] trt -> por cliente tb  o valor total do frete é DESPACHO + PEDAGIO + TAS + FRETEVALOR+ GRIS+ FRETE PESO
- [x] falta criar o botao de ação no portal de vendas pra cotar frete nos pedidos de tabela de frete, tem que esperar a resposta do erick se vai ser todas as transportadoras ou ele vai setar a transportadora

-> TRAVA DE PEDIDO
SE NAO TIVER SELECIONADO A LINHA JA INSERIDA NA COTAÇAO FRETE, AI SIM EXCLUI E INSERE

69004 - ATIVA
59535 - TERMACO

26/07 - 5 
29 2
30 6
31 8
01 2
02 5
06/08 8
07/08 8
08/08 5
09/08 -8
12/08-8
até o dia 12/08

ATIVA -> DEU CERTO, A CASA SUBIU PRA CIMA EM ALGUNS VALORES -> PEDIDO 61890
TERMACO -> DEU CERTO, MESMA COISA DA ATIVA, EM RELAÇÃO AS CASAS DECIMAIS! -> PEDIDO 54400
PAJUÇARA -> OK! (fiz na mao pra bater a tabela com cte bateu! nao tinha cte em teste)
BERTOLINI ->  NAO BATEU AS TAXAS FIXAS, PEDAGIO E ETC, ACHO QUE É TABELA DESATUALIZADA! -> PEDIDO 72239

SELECT INT.*,TX.NOMETAXA,CAB.PESOBRUTO FROM GMITFREINT INT
INNER JOIN TGFCAB CAB ON CAB.NUNOTA=INT.NUNOTA
INNER JOIN GMITTX TX ON TX.CODTAXA = INT.CODTAXA
where CAB.nunota =54400;

colocar frete valor na aba FRETE! -> OK

ver calculo ICMS -> email bertolini -> OK

FALTA VER OS ULTIMOS EMAILS DO ERICK, ELE MANDOU UNS CTE DA PAJU ACHO E BERTOLI -> OK, BERTOLINI NAO. -> OK

comparar pedido ativa com pajuçara -> fazendo a dash pra comparar 61890 -> OK

---

SÓ FICOU PENDENTE A QUESTAO QUE A BERTOLINI NAO USA O PESO BRUTO E SIM UMA PESO CALCULADO EM CIMA DE UM M3, QUE NAO SABEMOS DA ONDE VEM.

13/08

- [x] fazer no mes Anterior uma linha de saldo de todos os meses anteriores!
- [x] ver acesso ao registro amostra pq alguns usuario estao aprovando produto acabado
- [x] PEDIDOS X FATURADOS -> O VALRO DO FATURAMENTO QUE TIVER PEDIDO DENTRO DAQUELE MES DE NEGOCIAÇAO E QUE A DATA DE FATURAMENTO É DENTRO DAQUELE PERIODO
	- [x] E QUE TIVER PEDIDO NA TGFVAR!
- [x] AJUSTAMOS O TIPO NEGOCIAÇAO LINK DA CIELO -> NUNCA TINHA SIDO FEITO PARCIAL EM CIMA DO LINK -> E QUANDO TEM VALOR DE ENTRADA TAVA DANDO ERRO PRA FATURAR PQ O VALOR DE ENTRADA ERA MAIOR QUE O VALRO DA NOTA!


APAGAR OS DUPLICADOS NA CONCILIAÇÃO:

  SELECT * FROM TGFMBC WHERE DTLANC='01/08/24' AND CONCILIADO='N' AND HISTORICO = 'LPW' AND CODLANC=1 AND NUMDOC IN (SELECT NUMDOC FROM TGFMBC WHERE DTLANC = '01/08/24' AND NUMDOC IS NOT NULL AND CODLANC=1 AND NUMDOC <= 30864 GROUP BY NUMDOC HAVING COUNT(NUBCO) = 2);
            
            SELECT NUMDOC FROM TGFMBC WHERE DTLANC = '01/08/24' AND NUMDOC IS NOT NULL AND CODLANC=1 AND NUMDOC <= 30864 GROUP BY NUMDOC HAVING COUNT(NUBCO) = 2;



20/08

inclusao na TGFCUS pra movimentaçao interna de um produto

alteraçao na dash venda por periodo -> tiramos algumas TOP's! 1116 1190


21/08

ACOMPANHAMENTO DIARIO CONTAS E REMESSAS -> alteração 

	Saldo Carteira Simples -> e colocar SUM 
	Saldo Descontada à Vencer
	Tabela só das descontadas separada

Saldo Descontada à Vencer


RENATO ERRO NO MPS 42 - SEMANA 1

arrumamos o card de notas pendentes de correçao tirando oq tava com status nfe de enviada apenas.

Fiz 2 cards : Pedidos A Confirmar e Notas A Confirmar também!

E adicionamos algumas colunas a mais na dash de Pedidos x Faturados -> pra bater o saldo dos pedidos em aberto!

22/08

excluindo financeiros sefaz porto

23/08
COLOCADO FILTRO PADRAO NA TELA MOV FINANCEIRA!




----

- [x] alteração na ext -> uma tabela intermediara na Frete -> aonde vai ter Data Vigente!!!
	- [ ] GMITCFGF,GMITCFGFR,GMITCFGTX,GMITCFGTXC,GMITCFGFRV,GMITCFGFRC
	- [ ] -> nas bases vai ter que dropar antes de rodar o WPM DNV, teste ja foi feito!
	- [x] testar GmIntTranspCfgTaxaCli -> se tiver um cliente especifico nao faz pra outro cliente a generalidade!
- [x] ver se as libs externas nao tem que por em outro lugar!


28/08

- [x] ver os cards de valor de pedidos da ju, pq nao estamos desconsiderando corte! fiz só nos de pedidos.
- [x] se tiver DTPREVENT preenchida nao pode faturar na data ou pra frente.
- [x] na administração de pedido -> colocar data previsao de entrega.
- [x] feito campo calculado de quantidade negociada!
29/09
- [x] card do financeiro Pagamentos - Em Atraso. e Pagamento - Em Atraso SEFAZ!
- [x] finalizado dash do Pedidos x Faturados -> agora com saldo acumulado
- [x] nos detalhes do cards -> abrir fin na linha e ordenar por data de vencimento! numero da nota e empresa dos financeiros detalhes
- [x] fazer card do top 5 menores e maiores!
- [x] ajustar o card da produção 


Colocar TIPVEND = V na confirmaçao pedido !? -> só no insert!

e ver financeiro lá da comissao que nao entrou na view mais!



05/09
ajustado um pedido com frete e tipo negociaçao cielo -> gerou o financeiro carteira
alterado  a rotina de confirmaçaoPedido pra Venda tb!


10/09
vimos lotes aprovados nao estando STATUSLOTE = 'P' na TGFEST!
produtos com estoque nas empresas nao conseguindo faturar!