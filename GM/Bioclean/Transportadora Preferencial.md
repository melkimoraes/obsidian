transportadoras por estado e cidade


transportadora -> estado -> cidade
	       -> clientes especificos


caso tenha somente estado sem cidade -> vai atualizar os cadastros dos clientes que tem esse estado no cadastro
se tiver cidade -> só vai atualizar o cadastro de clientes dessas cidades especificas dos estados!!

caso outra transportadora tenha o mesmo estado -> preciso atualizar somente o cadastro dos clientes que nao sao daquelas cidades setadas dessa outra transportadora!!

cliente especifico é um plus mais -> pra atualizar aqueles clientes!!
CASO o cliente seja de outro estado e tenha esse estado em outra transportadora, nao pode atualizar o cadastro desse cliente pq ele já é especifico dessa transportadora!!! -> OK


botao de ação na tela pra atualizar -> PROC -> OK -> STP_U_TGFPAR_TRANSPPREF_GM
posteriormente fazer uma trigger -> before insert vendo se tem algo na tela pra setar a transportadora preferencial! -> OK -> TRG_I_TGFPAR_TRANSPPREF_GM
fazer trigger pra nao poder inserir cliente especifico que ja ta em outra transportadora e nem cidade que ja tem em outro estado.
	-> TRG_I_TRNSPPREFC_GM
	 -> TRG_I_TRNSPPREFEC_GM
	 -> TRG_I_TRNSPPREFP_GM -> excluida


CODTIPPARC -> perfil principal



-> Perfil vai ter que ser uma aba dentro de estado
	-> caso tenha um perfil lá eu vou filtrar pra atualizar esse perfil naquele estado/cidade.
	 -> perfil pode se repetir
	 -> oq nao pode é ter estado/cidade iguais pro mesmo perfil dai! mas acho que dai nunca vai acontecer pq ja tamo barrando a inserçao de cidade repetida!