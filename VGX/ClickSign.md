empresa de telecomunicação - telemarketing

https://developers.clicksign.com/docs/buscar-modelo

vou selecionar 100 colaboradores e enviar pra API do ClickSign


kit admissional ou kit de contrato
kit férias

tela funcionários -> botão de ação

vai selecionar os candidatos -> seleciona os contratos


criar admissao
	-> tipo do envio, whatsapp/email/sms
	-> modelos de contratos por empresa diferentes!
		-> modelos do contrato é justamente a chamada do modelo que ta dentro do clicksign
	-> nome da admissao (nome do lote que vai criar no clicksign)
	-> hoje no sistema antigo, eles nao selecionam varios funcionarios, na vdd eles tem um arquivo .xls que tem CPF e MATRICULA que dai faz o depara no sistema deles pra enviar essa solicitaçao via API do ClickSign


criar envelope
colocar documento no envelope -> via modelos ou novo documento (que seria um arquivo vind odo sankhya)
colocar signatario no envelope
disparo de notificaçao de envelope ou signatario!


-> eles querem selecionar um botao na tela de Funcionarios pra gerar documento pra esses funcionarios selecionados.
-> e também apartir da tela de pré-candidatos -> Cadastro Pré-Candidato
	-> entao apartir daqui também o botao pra gerar os contratos.


cliente quer que seja automático a seleção dos contratos
	-> vai ser o excel que a Thais -> diferenciação + produto vai ter essas informações tanto na Funcionários como na Cadastro Pré-Candidato -> ai vou fazer depara pra selecionar o contrato!


vai ter contrato adicionais que vão ser criados em Sankhya. -> aviso previa, aviso de ferias vao precisar de assinatura.
-> aonde vai ser o melhor lugar pra selecionar os funcionarios (varios) pra envio desses relatorios
	-> acho que o ideal vai ser criar uma dash com base na tela Funcionários -> só que com filtros de setor/empresa pra ser mais facil multiplos funcionariso e e enviar esses relatorios via clicksign


---
vai ter que ter uma tela de Documentos
	-> descrição do documento
	-> seja um documentos pra varios arquivos(relatorios formatados ou modelos clicksign)

vai ter que ter uma tela de Modelos ClickSign -> pra ter código desses modelos e descrição, pra inserir na tela de Documentos. 
	-> inclusive aqui eu posso criar alguns botoes de açao que consigo resgatar os modelos do clicksign pra dentro dessa tela.

botão de ação -> tanto na tela de Funcionarios como na tela Cadastro Pré-Candidato

mas vai ser ideal criar uma 

tela Integração ClickSign -> tela de acompanhamento geral da integração


Telas:
Integração ClickSign -> tela de acompanhamento geral da integração
Documentos
	-> descrição do documento
	-> seja um documentos pra varios arquivos(relatorios formatados ou modelos clicksign)
Modelos ClickSign -> pra ter código desses modelos e descrição, pra inserir na tela de Documentos. 
	-> inclusive aqui eu posso criar alguns botoes de açao que consigo resgatar os modelos do clicksign pra dentro dessa tela.
Contratos Automaticos -> com base nas informações de experiencia, diferenciaão e optante ou nao VT -> quando selecionar o funcionario e clicar em gerar contrato clicksign nao precisa selecionar um documento especifico!

Dashboards:
dash de Acompanhamento ClickSign -> pra mostrar por status todos os documentos gerados no ClickSign pra acompanhamento.
	-> aqui vai ter as opções de aprovação/desaprovar -> o contrtao gerado, como segunda checagem do processo.
	-> aqui ter opção de cancelar envelope
dash também pra ter melhores filtros pra selecionar os funcionarios somente e gerar esse envelope clicksign. 


botão de ação -> gerar documento
-> na dash de Funcionarios que vai ser criada, como citado acima.
-> nas tela de Funcionarios como na tela Cadastro Pré-Candidato (telas que ja existem no sistema.)


