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
- [ ] ver PALETE CHEP
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
- [ ] FILTRO DE DATA DE CHEGADA PROTOCOLO DE ENTRADA E SAIDA.
- [ ] rodar amanha: UPDATE TGFPRO SET CODPARCFORN=1 WHERE ATIVO='S';
- [ ] fazer evento no libconf=’s’ pro pessoal da contabilidade!
- [ ] falar com o Bruno → travar ordem de carga em entrada??
- [ ] FAZER A COPIA DE ESTOQUE NA ATUALIZAÇÃO TABELA DE PREÇO
- [x] cancelar inventario → 


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