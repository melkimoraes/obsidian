	- [x] fazer trigger na AD_TRANSFLOCID → verificando STATUSETQ = ‘COMPRADO’ na AD_TGFIDETQ
- [x] colocar a query da auditoria de compras em produção
- [x] ver transferencia de palete caixa pra SRV que vai ter venda pro cliente.
- [x] fazer um relatorio formatado → Contabilidade → tops que baixam estoque(Almoxarifado) → nao confirmadas.
	- [x] 1339 → ATIVAR AÇÃO AGENDADA
- [x] ver java de imprimir etiquetas clicheria.
- [x] OC com nro pedido zoado
- [x] erro nas entradas de mesmo produto, mesmo lote porem separados.
- [x] ver repositorio de arquivos → /Livros Fiscais/TRIAGULAR.txt
- [x] kardex da AD_MAPAPRODINVT
	- [x] TRG_IU_MAPAPRODINVT_ART → VER AMANHA COM  O GABRIEL
- [x] clicheria → report OS
- [x] atualização preço de venda → % negativa
- [x] Luis custos → 
- [x] fazer botao de ação protoclo de entrada → laudo de lenha
- [x] COMERCIAL CHAPA
	- [x] valor negativo na tabela de preço
	- [x] valor positivo na tabela de preço
- [x] ver amanha com a Giovanna → email da Bet de divergencia da OC com nota de compra
	- [x] e validar dash de notas nao confimradas
- [x] Dash Estoque Almox igual da auditoria de custo porem uma tabela só agrupando por endereço!
- [x] separar por nota por aplicação → pra cada aplicação uma TOP. → TRG_I_TGFTOP_TOPSUBSTIT_ART, PROC_REQCMPAPLC_ART
	- [x] 400 → uso e consumo
	- [x] criar a TOP produtos intermediario → Silvia vai criar a TOP. 
	- [x] amanha testar a ação agendada Rateia por Aplicação Requisição da Analise de Giro → em producao! 6565 E 6755 TESTAR
- [x] NOTAS DE AJUSTES DE ESTOQUE → COLOCAR USUARIO INCLUSAO/ALTERACAO
- [x] criar ação agendada pra confirmar requisições 515 automaticamente.
- [x] fazer documentação do rateio da requisição feito pela analise de giro
- [x] aprovar ação agendada 1342 → testar se aprovou as notas 515 do chep!
- [x] fazer re-transferencia pro recebimento do armazenado → com o gabriel!
	- [x] ver com o gabriel baixa de requisição das tintas. quando sem endereço. → posso fazer uma trigger pra setar o codlocalorig sempre DEP. DE TINTAS.
- [x] fazer documento de devolução + exclusão de armazenamento(ou transferência?)
- [x] requisição de compra clicheria → empresa SRV nota 843 → CENTRO DE RESULTADO DE SRV tanto serviço/produto
	- [x] ABA ORDEM DE SERVIÇO(FICHA DE FACA) É A TELA REQUISIÇÃO → VER COM O MINUTTI MATAR OS PROBLEMAS DA EMPRESA.
- [x] ver a dash de requisição de produto → ver as tops novas.
- [x] colocar vlrunit com vlr do custo médio nas transferencias → Gabriel
- [x] ver PALETE CHEP
- [x] TESTAR A TOP 515 SE VAI TIRAR DO ESTOQUE DE TERCEIRO E ENTRAR NO PROPRIO.
	- [x] TALVEZ VOLTAR A CONFIG DA TOP 515 PRA ENTRAR NO ESTOQUE PROPRIO DNV E  DAR UPDATE NAS ULTIMAS NOTAS!
- [x] VER FINALIZAR OP !
- [x] VER RELATORIO FORMATADO
- [x] copiar analise de giro reis → tamara?
- [x] fazer copiar tabela de preço e selecionar varias tabelas e reajustar % reajuste.
	- [x] perguntar pra ela se ambas as personalizações sao somente de preço
- [x] ver embalagens de paletes que nao geram remessa! 
	- [x] e ver paletes que geram remessa → as notas modelo
- [x] alterar apontamento de caixa SRV → pegar o IDIPROC com base no numero do pedido e a sequencia
- [x] FILTRO DE DATA DE CHEGADA PROTOCOLO DE ENTRADA E SAIDA.
- [x] falar com o Bruno → travar ordem de carga em entrada??
- [x] cancelar inventario → 
- [x] REQUISIÇÕES LIBERADAS NAO CONFIRMADAS.
- [x] ARMAZENOU O AMIGO EM TO E FOI EM KG?
- [x] NOME DO USUARIO NO LOG INCORRETO
- [x] ANALISE DE GIRO
- [x] ERRO NA HORA DE FINALIZAR REQ
- [x] chep → em SRV talvez o parceiro que vai baixar é do FIFO pq lá a nota entra como ITA


## Important 🛑

- [x] rodar amanha: UPDATE TGFPRO SET CODPARCFORN=1 WHERE ATIVO='S'; → SNK_MATGIRCALCSUG_EXT
- [x] FAZER A COPIA DE ESTOQUE NA ATUALIZAÇÃO TABELA DE PREÇO
- [x] fazer evento no libconf=’s’ pro pessoal da contabilidade!
	- [x] criar preferencia PERC LIB
	- [x] criar marcação nas TOP que vai criar evento 6868
- [x] VER AS EXC
- [x] 5261 - produto nao tava na analise de giro
- [x] COLOCAR A VALIDACAO AD_ENTPLT → CAMPO DA TOP NO JAVA DO PALETEREMESSA
- [x] TESTAR TRANSFERENCIA DE CHEP → APP
- [x] QUANDO CONTAR CHEP → MUDAR A TOP 515 NOVAMENTE
	- [x] TIRAR A VERIFICAÇÃO DO ESTOQUE DO PALETEREMESSA (BOOLEAN QUE FIZ PRA VERIFICAR)
- [x] VER EDI PALETE CHEP → TA CONSIDERANDO DEVOLUÇÃO?
- [x] VER AMANHA COM FILIPE/IGOR → COMPRAS PODE COTAR MAS ACREDITO QUE PRA GERAR O PEDIDO DAR O ERRO OU LIBERAÇÃO?
	- [x] STP_RGN_COMPRAHOM_ART
	- [x] VER SE VAI MARCAR OS PRODUTOS OU TOP.
- [x] VER A SOLICITAÇÃO DO VITOR DE CANCELAR VARIAS ETIQUEATS, HOJE PELA LIXEIRA.
- [x] ver devoluções incorretas do chep
	- [x] fifo
	- [x] arquivo remessa se vai pegar oq eles ja tinham feito
- [x] quando Igor retornar os produtos pra passar pelas validações de homologação → explicar e liberar o evento 1500!
- [x] colocar os PP pra finalizar caso PA finalizado
- [x] natureza do palete CHEP → ver com o contabil → nao pode ser a msm natureza da nota de debito
- [x] LANÇAR REFUGO, COM ERRO → ELTAO
- [ ] ELTAO BOTAR NA EXT FILTRO DE DATA CHEGADA NO PROTOCOLO DE ENTRADA
- [ ] ver com o bruno talvez uma tela de pedidos agil → comercial?
- [ ] ELTAO → JAVA DO FINALIZAR TRANSFERENCIA SAIDA BOBINA → COLOCARMOS PRA COLOCAR VALOR ICMS NA NOTA DE COMPRA QUE DUPLICA APARTIR DA VENDA
- [ ] Eltao → desabilitar botao de excluir composição
- [ ] Elton → travar o campo Vendedor no PRECALCULO
- [ ] Elton → erro no anexo RD.
- [x] Elton → Colocar filtro de ATIVO = ‘S’, Cores da TGFPRO (RD)
- [x] ver fc dos vendedores → marcação de tipo assistentes(acho que nao é esse o nome) e se for vendedor tb ver a aba das assistentes mas com data de validade(pra caso deferias)
- [x] Apontamento de Acessórios -> quando alterado peças por pacote -> alterar peçar por palete
	- [x] acessorios@artivinco.com.br
- [x] Travar o desmembramento de etiqueta caso não for o pedido todo!!
	- [x] top de destino, nao tiver na top que gera cota maquina → nao trava
- [x] fazer a melhoria na tela de apontamento de refugo 2.0 de gerar 1 nota seja de entrada ou saida pra todas as etiquetas!
	- [x] STP_GERAAPOREF2_ART E STP_GERAAPOREF2_CABITE_ART
- [x] AVISO OU TRAVA -> valor do item no pedido de venda ao salvar -> se estiver data de entrega dentro de um período de PC Aprovado e o valor estiver diferente deste travar/avisar.
	- [x] botão de ação prorrogar na alteração de pedidos.
	- [x] STP_U_TGFITE_PRCO_ART
- [ ] Fazer a melhoria do gerar apontamento em java!
- [x] 296 -> TRANSFERENCIA DE LENHA -> GERAR ENTRADA
- [x] ficar de olho nas notas da lenha do dia 8 em diante
	- [x] ver como vai entrar a nota no almox, sera que vai ter que por pra conferir?
Gabriel
- [x] feito a trava TRG_I_ITE_VALESTAPP_ART → das tintas somentes na 500 e 700 porem no APP nao retorna erro ao tentar confirmar a transferencia!
- [x] produtos com top 700, dias diferentes só que com vlr unitario 1 e outro com valor, mesma top e empresa.
- [x] a nota de lenha era pra entrar na conferencia APP? nao conseguimos ativar isso pois nao tem pedido!
	- [x] e quando eles conferem pra menos? moraes falou que não tem necessidade de fazer uma devolução
- [x] VER COM O GABRIEL, A TRANSFERENCIA DE PALETE QUE O PEDRO PERGUNTOU
	- [x] QUANDO TERMINA DE TRANSFERIR ERA PRA MUDAR A EMPRESA DE FATURAMENTO NO PEDIDO ORIGINAL?


---

CHEP

a nota intermediaria do elton antes da emissao da nota do chep → só tira do estoque de terceiro?
nao tem nota de entrada!

ENTRAR COM UMA NOTA NO DIA 09??

nao ta sendo gerada a nota 220 apartir da 255

se gerar a nota 220 → talvez aquela nota intermediaria que o elton emite que tira do terceiro e da transferencia do app também nao devam ser efetivadas.

204 → 109276
412

custo da chep e do muller → como fazer?

---

- Zerar o estoque fevereiro dia 8 do papel → faz a copia e pega a CTE. empresa 2.
- Empresa 1 vai limpar as notas de papel. anterior a 08/02.

--RODAR A COPIA DE ESTOQUE DO DIA 08/02 DESSES GRUPOS DE PRODUTOS
SELECT DISTINCT CODGRUPOPROD FROM TGFPRO WHERE CODPROD IN (
SELECT DISTINCT CODPROD FROM TGFITE WHERE NUNOTA IN (
SELECT NUNOTA FROM TGFCAB WHERE DTENTSAI <= '08/02/2025' AND CODEMP = 2) AND CODPROD IN (SELECT CODPROD FROM TGFPRO WHERE DESCRPROD LIKE 'PAPEL %'));

EMPRESA 2
--5311003 - 2 NUIVT
--5311006 - 3
--5311007 - 4
--5312001 - 5
--5312002 - 6
--5312006 - 7
--5312007 - 8

-- 2043351 - 2043369

EMPRESA 1 : 
2043564 - 2043582
NUIVT- 9


---

220 -> testar reentrega

1030287

---

TROCAR OS LOCAIS DOS PALETES DE REMESSA
NA SELECT CODLOCALPAD fROM TGFPEM WHERE CODPROD=423;


---

C:\sk-java\DWFDesigner\jre