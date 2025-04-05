
	PEDIDOS NAO CONFIRMADOS SEPARAR EM DOIS 
		-> PEDIDOS PENDENTES DE LIBERAÇÃO
		-> PEDIDOS REALMENTE NAO FOI CONFIRMADO NENHUMA VEZ

FATURAMENTO COMERCIAL -> VALIDAR VALORES TOTAL LIQUIDO
	-> NAO ENTRA INTERCOMPANY, MAS NAO ENTRA A VENDA BIOCLEAN - 1126!?


DESCONTADA - colocar contas das descontadas!!!

CARTEIRA SIMPLES -> REMESSA PRO BANCO QUE ESTA EM CARTERIA SIMPLES -> total dos financeiros do pix is not null que nao tao baixados!!!

---
- [x] faturamento
- [x] subir ext quando der ajuste do IE da dlog!
- [ ] etiqueta que vai chegar ->10 LARGURA 15 DE ALTURA
- [x] ajustar o mapa de separação pra kits nro unicos pra testar: 28076 - 27531 - 27526
- [x] subir ext correios pros numeros unicos gerar etiqueta : 27522 e 27521
	- [x] 27521 -> gerei já e ver as ações agendadas se tao ok!
- [x] e fazer o gerar etiqueta automatico!! no notepad tem oq eu reescrevi mas nao testei -> testar na base local e gerar etiqueta jadlog! que da pra excluir!
- [x] finalizado dash de Sugestao de Compra + botao de ação criando pedido.
- [x] FILTRO DE REDE NA DASH DE Faturamento por Empresa - Fiscal Detalhado
- [x] adicionado coluna na dash STATUS DOS ITENS DOS PEDIDOS
- [x] dash FATURAMENTO POR VENDEDOR - adicionado filtro de marca,vendedor
- [x] ajuste nos relatorios de etiqueta que tinham a barra azul -> trocado pra preto
- [x] trocado nomedoparceiro pra razaosocial do parceiro nas etiquetas!
- [x] lista de financeiros - nufin, valor , PARCEIRO, TIPO DE NEGOCIAÇAO, TIPO DE TITULO, BANCO , se foi registrado no banco sim oui nao, se tem o NOSSONUM se tem o pix se tem deposito. DTFATUR das notas, só receita
- [x] refizemos a dash de Carteira Simples V2 -> mesma ideia da primeira, agora também com detalhe expadindo conta pela data de vencimento
- [x] erick quer um relatorio de etiqueta que coloque o codigo do volume só pra imprimir aquele
- [x] Dash Relaçao Pedidos x Faturados -> filtro da marca ter a opçao TODAS e filtro de TIPO se é Venda/Bonificação
- [x] Criação nova dash Abertura - Pedidos
- [x] alterado tamanho pra maior nas etiquetas NF, Volume e Nome do Destinatario.
- [x] Alteramos dash DASH RELACAO PEDIDOS X FATURADOS - AUMENTAR OS QUADRANTES DOS TOTAIS e COLOCAR FILTRO MES/ANO


P_COMINTER
Considera Intercompany?

AND ( FAT.GRUPO_TOP <> 'Intercompany'
          AND :P_COMINTER = 'N'
          OR NVL(:P_COMINTER, 'S') = 'S' )     


P_TIPO
Tipo
N=Venda
S=Bonificação
AND FAT.BONIFICACAO IN :P_TIPO


P_CANAL
Canal
SELECT OPCAO AS VALUE, OPCAO AS LABEL FROM TDDOPC WHERE NUCAMPO = 9999994136
UNION ALL
SELECT '<SEM CANAL>' AS VALUE, '<SEM CANAL>' AS LABEL
FROM DUAL
AND FAT.CANAL_VENDEDOR IN :P_CANAL



--------
COMERCIAL

Analise de Compras - FOI
- [x] criar campo - INDIRETA/DIRETA
- [x] FILTROS: PERIODO, STATUS DO GRUPO, NUMERO NOTA OU NUMERO UNICO, FILTRAR POR TOP, PARCEIRO, PRODUTO.


DASH -> PROCURA A MATERIA PRIMA EXPLODE OS PRODUTOS FINAIS QUE USAM OU PROCURA O PRODUTO FINAL EXPLODE OS PRODUTOS DA COMPOSIÇÃO.  - FOI

Relatorio de Entradas de Notas Fiscais -> FOI NA ANALISE DE COMPRSA

Relatorio de compras e devoluções -> quantidade , valor, e explodir as notas -> linkar com a ocorrencia da recusa do laudo de amostra. -> FEITO sem a ligaçao com ocorrencia da recusa do laudo por enquanto -> Relatorio de Devoluções

Relatorio de posição de estoque -> trazer ultimo custo de reposição -> ja tinha, só coloquei ultimo custo -> [DASH] Estoque Atual + R54 - Posicao de Estoque com Custo

Relatorio de pedidos -> pendente ou nao pendente, fornecedor -> FOI -> Relatorio de Pedidos - Compras

Relatorio de Saving -> percentual do ultimo custo e do novo e valor pra dar a diferença de saving!

relatorio de cliente e fornecedor -> soma por cliente! -> analise de compras - geral
	
	Relatorio de produtosque mais comprou ->> relatorio de produtos mais vendidos 
			 analise de compras - geral

**SOLICITAÇÕES DE COMPRA / COTAÇÃO** -> VER COM O ANGELO/ALLYSON


- [ ] criar trigger before insert na par CLASSIFICACAO ICMS -> erick -> FOI
- [ ] verificar monitor de consulta -> erro na ficha de parceiros -> FOI
- [ ] alterações transportadora preferencial -> perfil -> FOI

DASH DA JU
filtrar por empresa -> BIOCLEAN, BIOTESTE(501), DIMARO, PORTO, VIVAZ
FILTRO PERIODO DATA NEGOCIAÇÃO FIN
FILTRO CONTAS BANCARIAS -> 
peguei todas as receitas -> baixadas e pendentes

ME DA O VALOR TOTAL DE DESDOBRAMENTO
VALOR SIMPLES -> É BOLETO -> NOSSO NUMERO PREENCHIDO
DEPOSITO -> DIFERENTE DE BOLETO 
CARTAO DE CREDITO ->
CIELO -> 
SEM NADA -> 

NAO CONSIDERAR INTERCOMPANY -> falta tirar isso aqui


- [ ] campos calculados de cidade,estado,uf,regiao cliente na TGFCAB


23/07 ->

ver os emails htmlm bugado erick -> OK feito varios agendadores no relatorio formatado pra ficar por empresa.

ver observação da nota que nao foi pedido -> visto com o jader 

colocado filtro de parceiro e produto -> criado view pra respeitar usuario que ta pesquisando na dash

24/07 - 09:04

ALTERAMOS FC_OBSNOTAS NOVAMENTE -> COLOCAMOS A TOP 1126 NA MSM REGRA DA 1151.

Alteramos a trigger que estava travando Caso o financeiro estive com conta de recebimento diferente da conta setada na empresa! TRG_IU_TGFFIN_BCOCTA_BIO 

Alterando a ação na TGFCAB - Alterar Qtd. Volume / Tipo Frete -> tiramos a obrigatoriedade de preencher todos os campos e ficou dinamico a alteraçao dos 3 campos na TGFCAB

alterando etiqueta -> além daquela numeraçao de volume por remessa, ele quer o volume antigo totalizador das remessas!

alterar os pedidos quando alterar qtd na remessa!? -> nao fiz ALTERAR O 65978 PRA ALTERAR O 64496 -> feito no msm evento que eu fiz na TGFCAB pra alterar transportadora

anderson chamou nos whats -> ao gerar etiqueta correios tava dando um erro, mas ja avisei que colocamos uma mensagem melhor -> é pq tava sem serviço setado na nota

ver fc_obsnotas dnv!! -> erro meu , ja vi!

Colocamos Alterar Local e Limpar Lote na Administração de Pedidos.

angelo que ver algo no mps - !??


25/07 

JADLOG TROCAMOS O NUMERO DO PEDIDO PELO NUMERO DA NOTA AO GERAR ETIQUETA

criamos um by pass que esta na preferencia GMPARCTRANSPINT -> se o codigo transportadora for igual desse parametro, nao validamos se transportadora ta igual do cotado ou do pedido!

e criamos trava na cotação de frete (extensao) -> se ultrapassar o peso limite da preferencia GMPESOLIMITECOT, dá um erro pedindo pra selecionar uma transportadora mais especifica pro cliente, e nao vai cotar via api!

algo sobre considerar devoluçao -> VIMOS A VW_PEDIDO -> fizemos filtro de marca pela TGFMAR agora, também fizemos varios filtros pra filtrar TOP

sobre a tabela de frete 


---

- [ ] trava na tela do MPS -> se tiver MRP gerado nao permiti alterar!

