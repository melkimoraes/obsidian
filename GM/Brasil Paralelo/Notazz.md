API_KEY -> 
wY1UTMmFTYxkjZxEzN3kTYyETOhhDZjVTN4QmM5cDM2oHd14ke5QVMuNzN2czN3EjYiJmNyUTMlRTZjF2NwEGOwMGZilDM2QzMnZ

[https://app.notazz.com/docs/api/#introducao](https://app.notazz.com/docs/api/#introducao)


Whatsapp - 37 9981-7657 -> contato , menu de opções desenvolvimento  -> codigo 1


- [x] campo na TGFPAR - AD_INTNTZ
- [x] tabela de CONFIG + INTEGRACAO
- [x] campo na TGFCAB - AD_IDNTZ
- [x] preferencia - INTNTZTOKEN
- [x] nao testei o gerar nota!!
- [x] achei um método do cancelar do padrao -> porem a nota ja ta cancelada na sefaz entao ele nao faz a operaçao de inserir na TGFCAN e etc!
- [x] o mesmo vale se eu tentar deletar a nota! Ele fala que ela está aprovada!
- [x] falta criar o campo da GURU AD_IDPED, nao tem mais nas bases!
- [x] falta fazer o doc!
- [x] FIZ UMA DASH !
- [x] limpar do dia 02/07 pra traz a base de teste!!!
- [x] falar com o william
		vou tentar na mao processar -> fica no aguardo ação agendada
		tanto nfe, nfse não traz nada se é estrangeiro um código -> eles nao tem esse codigo!
- [x] dar update nas tgfite pro CFOP igual nas notas aprovadas que nao tiverem!
- [x] integrar dia 01/07 - 03/07 , 13/07 - 14/07, 16/07 -> gerar notas de todas elas também e terminar dia 12/07
- [ ] sobre a evidencia da carol que nos dias 01/08 a 05/08 no notazz mostra 36 notas canceladas, depois de falar com o wiliam e tentar aquela alteração o teste que fiz deu 3 notas canceladas somente, ver com ele se é algum filtro que ela ta fazendo errado no notazz! ou se era pra dar mais também na alteração que fiz.
	- [ ] do jeito antigo deu 11 notas canceladas
- [ ] outra coisa é ver com o wiliam é se devo fazer essa alteração pra notas canceladas de produto!?







- [x] ja joguei na base de teste, mas nao fiz lançador e nem joguei o java!

PRAS NOTAS NFSE
	Nro NFS-e -> MSM NUMERO DA NOTA PREENCHER. 🆗
		TALVEZ SERVIÇO TENHA QUE PREENCHER  Cód. trib. ISS

PRAS NOTAS DE NFE:
	serie na NFE -> serie tb na nota 🆗
	
	na tributaçao preencher codigo ICMS -> ISSO NA TGFITE
	
	INSERIR NA TGFDIN
	PIS - CODIGO 6
	COFINS - CODIGO 7
	
	
	PREENCHER A BASE DE CALCULO COM O VALOR DO PRODUTO
	Cst/Csosn - COLOCAR O CODIGO DO XML



0825


- [x] passar pra elas , caso de ajustar cadastro e etc, como processar as integraçoes? explicar pra elas!!
- [x] testar nota cancelada -> caso ja estiver contabilizada
- [x] recalcular pis/cofins! -> vai fazer rotina java? vai colocar 

30/07

- [x] colocar validaçao se a nota ja existe -> numero de nota, serie -> talvez pras notas eletronicas colocar serie chavenfe!
- [x] excluir as duplicadas.
- [x] ver o nota cancelada 29/07 
- [ ] inserir notas canceladas que nao tiverem a aprovada em sankhya -> pra cancelar.
- [ ] dia 15 nao tem nota duplicada, 9199 é a quantidade de linhas de notas que o notazz integrou.





campo Cidade -> na NFS → feito, verificando principalmente se tiver o campo preenchido no xml e também se o  CEP tiver preenchido ou for diferente de 99999999
	NA NFE -> campo cMun, → feito
-> verificação código municipo fiscal ambas os tipos



-> se na nota fiscal de serviço eu não tiver cpf, vou por EUA → ok
	-> na nota eletrônica -> eu vou verificar se o Pais tem nota eletrônica está correto no cadastro Sankhya. → ok



Devolução -> só nas notas de NFEA
-> criar campo de TOP Devolução -> Config Notazz -> top 1200 -> ver o tpNF = "0" na devolução.


---

BP PRODUCAO~1

TRG_TSICID_SP_GM