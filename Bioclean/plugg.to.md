
1023

https://developers.plugg.to/reference/getting-started-with-your-api
https://plugg.to/marketplace-de-integracao/integrar-com-beleza-na-web/

shopee
mercado livre
mercado livre full -> só o full emite nota
b2w
cnova
magalu
olist
drogaraia
beleza na web

ação agendada -> na plugg.to vendo se tem pedido pra integrar em sankhya!

oq o sankhya vai ter que integrar na plugg.to -> preço e estoque?

preço
	-> no antigo sistema era usado a tabela de preço VTEX e eles colocavam uma porcentagem maior em cima pra vender na plugg.to
	preço normal tanto os marktplace como o full!

estoque
		-> tb atualiza tudo na plugg.to -> tem o local proprio no sankhya ja E-COMMERCE.
		-> muda pro mercado full ! pq ele vai pra estoque em poder de terceiro! 
	O ESTOQUE DOS MARTPLACE a gente atualiza
	O FULL ele atualiza em sankhya!

frete
	-> a plugg.to emite as etiquetas pras transportadoras devidas do marketplace.


tudo na empresa 4 somente.

Ações Agendadas:
	- Busca de pedidos na plugg.to
		- Se for full -> nao é pedido e sim nota!
	- Atualiza Preço dos produtos na plugg.to! 
	- Atualiza Estoque dos produtos somente nos MKTPL da plgg.to
		- Se for full -> atualiza o estoque no sankhya, do estoque de terceiros.

Tabelas:
	- Depara de MTKPLACE
	- Depara de Produtos/MTKPLACE?
	- Depara do Frete


plugtoo -> ele quer usar cotação de frete, gerencia preço lá, varias coisas que na vtex nao tem!

---

Mapeamento Principal
	- Produto
		- vai querer selecionar o produto do sankhya e enviar pra plugg.to -> e dai quando inserir retornar o id plug.too em sankhya também.
	- Marktplace -> o marktplace -> vai ter que importar 
		- e varios campos que vamos usar pra tabela de preço, tipo de negociação, empresa, vendedor.
	- Preço
		- o preço vai ser por marktplace, entao no sankhya vmaos ter uma tabela pai PLUGGTO e dai as filhas por marktplace!
		- Ação agendada pra atualizar o preço de tempo em tempo.
		- ter um forçar preço
	- Estoque
		- Ação agendada pra atualizar isso de tempo em tempo.
		- estoque da 199 pra todos os marktplace.
		- pra kits -> pegar o menor estoque componente do kit, esse é o estoque do kit.
	- Pedido
		- Forçar o pedido, e um numero de pedido.
	- Fullfilment
		- Vai controlar o estoque de terceiro desse cara. Vai pegar a nota de remessa que eles geram no mercado livre, e dai essa nota em sankhya vai tirar do estoque e vai pra estoque de terceiro.
		- controlaar as notas de devoluçoes do mercado full -> pra voltar no estoque
	- Etiqueta
		- quando forçado o pedido a gente capturar a etiqueta pra eles imprirem em sankhya.
	- Tipo de negociação -> se for padrao da plugto sem ser por marktplace -> vou fazer uma tela pra fazer depara.

- [x] Etiqueta -> eles enviam ela emitida e enviam a etiqueta impressa?
- [x] VERIFICAR NO PROCESSAR PRODUTO → prodVO.asString("AD_DESCRPRODECM") SE NAO ESTIVER PREENCHIDO! ESSE NOME COMERCIAL É OBRIGATORIO
- [x] Como anexar o produto aos marketplaces -> é pela tabela de preço?
- [x] Cadastro de produto já pede um preço e preço especial fora do cadastro do produto??
- [x] Oq diz pra plugg.to que quero parar a venda de um produto?? Tipo o arquivar!? mas nao tem algo na API deles que fala como arquivar, sei que eles pedem pra zerar estoque
	- [x] testei o campo archived como true e zerar os estoques! → nao acatou o envio na API, o arquivar pode ser uma forma de desativar todo o produto , inclusive na pluggto ele exige zerar todo o estoque antes!
	- [x] fazer o teste, mas se eu quero cancelar a venda em um marketplace, basta eu sumir com a tabela de preço!?
	- [x] inclusive não pensei aonde jogar o desativar produto pra um marketplace especifico!
		- [x] vi como exemplo o mercado livre o enviar produtos (novo anuncio direto no marketplace) → mas isso na tela da pluggto!
- [x] Perguntar do produto kit → se eles colocarem um kit PK na tela eu vou ler e entendo que é um kit, vejo se os componentes estão cadastrados no pluggto e mando inserir o kit!?
- [x] Imagens → precisa ser url → ver com o erick
- [x] Nas fotos, preenchi o externla mas nao o url!
- [x] testar se o shippment id vai ser preenchido em um pedido teste e entao da pra testar o faturamento! e começar o schedule!
- [x] transportadora company tem quando busco o pedido sozinho!!
- [x] colocar ações agendadas de busca de pedidos e também processamento de pedido!

---


Exceção do produto → 
	DIMENSOES EMBALAGEM → MANDAR VAZIO 🆗
	Origem do Produto → Procedência Nacional → criar campo na tela de produto plugg.to 🆗
	Criar todos os campos que estamos subindo como cadastro do produto! os que puxamos do cadastro do produto sankhya, deixa somente leitura  e o de origem como é um campo especifico plugg.to deixa pra preencher obrigatorio! 🆗
	Imagens → TESTAR AS ABAS DE ANEXOS DO CADASTRO DO PRODUTO 🆗

nas ações agendadas de atualização de preço e estoque assim como no processar do produto → só substitui o log do estoque/preço caso tenha alterado esse valor! 🆗

fazer na extensao
- [x] colocar nro pedido pluggto na TGFCAB -  PLUGGID
- [x] colocar campo parcela pluggto na TGFCAB! - PLGTPARCELA
- [x] colocar marketplace na TGFCAB - PLGTMKTPLC
- [x] testar o ETIQUETA agora no pedido!
- [x] antes de colocar a ext na bioclean → a busca das informaçoes nos produtos voltar ao que era a busca dos campos na bioclean!
- [x] inserir etiqueta! → 16789524657 ou 672d085cb1b02a08b014d6b1
- [x] quando subir em produção a ext → criar tipos de negociação com parcela!!
- [x] criar layout do pedido de venda pra top plugg.to
- [x] talvez criar um processar pedido → com um pedido selecionado na  tela de PEdidos! → com isso vai ter que reescrever o prcoessar que esta como action/schedule
- [x] talvez também criar um forçar busca de pedido! passando o id!?
- [x] na tabela de Pagamentos Pluggto → colocar CODPAG de vez de ID!
- [x] view na tela de Martetplaces → com base no codigo da tabela de preço → dar o produto e o preço. colocar lá o ultimo estoque também.
- [x] A BUSCA DOS PEDIDOS → vai ter que por d-1 a data atual eu acho → o filtro que fiz traz 50 sempre!

to-do
- [x] exceção do preço dos produtos! → tava construindo pra por um union all no select, falto a parte das contas com o campo TIPO da TGFEXC! o duro que se for porcentagem vou ter que fazer conta com o valor de venda da tabela original!
- [x] desativação do produto! → testar o arquivar na plugg.to talvez descobrindo no postman qual campo diz respeito a isso!? se bem que se for desativar de um marketplace nao descobrimos
- [x] FAZER O KIT! 
- [x] ver primeiro o erro do produto já processado!
- [x] qualquer hora testar o erro que deu quando enviei varios produtos pra processar e um produto com erro fez nao atualizar o cadastro sankhya pluggto, coloquei o wrapperVO persist → só testar em teste com algum produto que nao tem estoque enviando junto com varios pra pluggto!
- [x] testar cadastro do cliente → e dai de quebra criar a aba cliente com as informaçoes! e a informaçao de entrega!
- [x] Talvez tirar Empresa da tela de Configuração já que vou usar nota modelo pra gerar o pedido!
- [x] Fazer tela de Log do processar pedido → fiz log direto no pedido, se caso precisar pode ser sim!! talvez o log pra outras açoes seja necessario
- [x] melhor colocar o Processar Produto, Processar Pedido como transação manual
- [x] Abas na tela de produtos como - Frete, Itens, Pagamento, Cliente
- [x] ver oq mais a Tati precisa na tela de Pedidos → nao coloquei informaçoes do cliente em especifico!
- [x] Sempre fazer uma consulta de pedido cancelado!? Colocar essa consulta na geração do pedido e na aprovação do pedido(faturamento)
	- [x] mas futuramente fazer um consulta de pedido cancelado → alinhar com o erick isso, pq talvez durante 7 dias depois do faturamento dos pedidos pluggto → monitor cancelamento.
	- [x] ou perguntar pra pluggto → até quando um pedido pode ser cancelado, ow se sempre podem ser
	- [x] O unico lugar que vai retornar erro ao tentar fazer algo com um pedido se estiver cancelado é no AprovaPedido → é aonde eu mexo no status!
		- [x] pensando nas rotinas:
			- [x] Busca → ja to buscando por status nao preciso ver se ta cancelada!
			- [x] Processa → era aonde estava a alteração pra aprovado, mas tiramos. → aqui talvez verificar status
			- [x] Aprova → vai retornar se cancelado porem, nao estou tratando nada, como por exemplo excluir pedido. → aqui vai ter que lgoar e cancelar a NFE!
			- [x] Gerar Etiqueta → nao sei se ao buscar label retorna cancelado, melhor tratar aqui também!
			- [x] a rotina dos 7 dias realmente é interessante → mas oq ela vai fazer!? excluir pre-nota? excluiir pedido?
			- [x] NA VDD VAMOS SOMAR ESSA IDEIA COM O ATUALIZAR STATUS DOS PEDIDOS ATÉ ENTREGUE!
- [x] Tem o Pedido Full ainda!
- [x] testar o confirmar pedido!
- [ ] Olist → quantidade de estoque pra eles é na geral da pluggto
- [x] Quando Tati gerar remessa no mercado livre → ver se essa nota aparece no portal xml.
- [x] acompanhar um pedido mercado full que vai cair!!
- [ ] Coisas que faltam + essa lista de cima:
	- [ ] Devolução do mercado full → ver como um pedido desse vai cair!? sera que é quando cancelar um pedido!?
		- [ ] essa devolução provavelmente o mercado full emite! talvez tenha que dar entrada nessa devolução em sankhya!!
- [ ] Fazer documentação da rotina!
- [x] continuar o cancelar no processar
- [x] ajustar atualização de estoque dos kits → pega o menor estoque de um componente.
- [x] ATUALIZAR STATUS DOS PEDIDOS → pros pedidos, vai sendo nas ações e depois da aprovação tendo o NUNFE → vai ter uma ação que vai rodar +7 dias da aprovação pra ver cancelamento! e dai logar isso no log do pedido!
	- [x] sendo como status final o Entregue
- [x] Dashboard dos erros de log dos pedidos 
	- [x] mais botão de ação pra limpar o log principalmente de cancelar um faturado!
		- [x] pro botao acho melhor criar aquela tabela de log
			- [x] → ela vai ter usuario e ação, possiveis açoes seria deleçao do pedido/prenota, e o log limpo dps da trativa!
- [ ] Ligar hoje o atualiza preço/estoque!
- [x] subir a dash em prod
- [x] subir a ext em prod
- [x] adiciona  ação agendada atualizar status!
- [ ] testar o atualizar status em produção pros entregues tb!! o status final
- [x] buscar pedido → do mercado full, ja vem como faturado, talvez separar pra filtrar aquele campo delivery_description + ack false
- [x] Testar o atualizar status com pedido cancelado!!
- [x] Segunda: ver pedido do full que nao caiu na pluggto!
- [x] botões de ação do pedido/produto → nao atualizar grid inteiro!!!
- [x] atualizar a PLGTPROD na volta do botao processar produto!
- [ ] futuramente:
	- [ ] mais filtros na tela Pedidos como: data, status
	- [ ] mais filtros na tela Produtos como: AD_TIPO
	- [ ] Promoções de preços.

1736

```
SELECT MKT.CODMKTPLC,
  MKT.DESCRMKTPLC,
  TAB.CODTAB,
  TAB2.NUTAB,
  EXC.CODPROD,
  EXC.VLRVENDA AS VLRVENDAORIG,
  ROUND((CASE WHEN NVL(TAB.PERCENTUAL,0) <> 0 THEN (((TAB.PERCENTUAL/100)*EXC.VLRVENDA)+EXC.VLRVENDA) ELSE EXC.VLRVENDA END),2) AS VLRVENDA,
  NVL(TAB.PERCENTUAL,0) AS PERCENTUAL
   FROM AD_MKTPLCPLGT MKT
   INNER JOIN TGFTAB TAB ON TAB.CODTAB = MKT.CODTAB AND TAB.DTVIGOR = (SELECT MAX(DTVIGOR)
FROM TGFTAB
 WHERE DTVIGOR <= SYSDATE
AND CODTAB = TAB.CODTAB)
   INNER JOIN TGFTAB TAB2 ON TAB2.CODTAB = TAB.CODTABORIG AND TAB2.DTVIGOR = (SELECT MAX(DTVIGOR)
FROM TGFTAB
 WHERE DTVIGOR <= SYSDATE
AND CODTAB = TAB2.CODTAB)
   INNER JOIN TGFEXC EXC ON EXC.NUTAB = TAB2.NUTAB
   WHERE MKT.ATIVO='S'
   AND EXC.CODPROD=748
   AND NOT EXISTS (SELECT CODPROD FROM TGFEXC WHERE NUTAB=TAB.NUTAB);
   
   
SELECT MKT.CODMKTPLC,
  MKT.DESCRMKTPLC,
  TAB.CODTAB,
  TAB.NUTAB,
  EXC.CODPROD
FROM AD_MKTPLCPLGT MKT
   INNER JOIN TGFTAB TAB ON TAB.CODTAB = MKT.CODTAB AND TAB.DTVIGOR = (SELECT MAX(DTVIGOR)
FROM TGFTAB
 WHERE DTVIGOR <= SYSDATE
AND CODTAB = TAB.CODTAB)
INNER JOIN TGFEXC EXC ON EXC.NUTAB = TAB.NUTAB
WHERE EXC.CODPROD=748;
```



---

PA2600
PA2785
PA2606
PA2468
PA2770



- inserir eles na plugg.to -> OK
- inserir tabela de preço - 10% beleza na web -> OK
- colocar dimensões embalagem na integração -> ESPERANDO RETORNO
- colocar categoria -> pegar perfil -> criar perfil plugg.to -> PUXAR DA CATEGORIA VTEX
- colocar descrição e-commerce. -> PUXAR DA TELA DA VTEX PRA CAMPO NA PLUGG.TO → OK

status do pedido → vai estar pendente, aprovar (atualizar na buscaPedido)
→ processando o pedido sankhya vai mandar o pedido + ack true lá
status do pedido → envia a fatura (talvez com o id do endereço!) → aqui verificar o status dele melhor!?
status do pedido → envia os dados do frete (caso for em sankhya) → mas no caso depois da fatura , vai ter que fazer o gerar etiqueta procurando aquele labels pelo id do pedido! pra retornar a etiqueta
depois status do pedido → entregue

uma vez o pedido aprovado → pode ser cancelado.

aprovado → faturado → entrega informada → enviado → entregue
aprovado → tentei aprovar noa foi
faturado → ao gerar NFe → pra enviar o faturado lembro que tive que ter o id do shipment!
entrega informada → nao achei, tem o shipped → porem é quando foi enviado pelo oq eu entendi
enviado → shipped! nao sei ainda quando vou atualizar isso, nao sei se vai ser atraves do rastreio dai
	→ A PLUGGTO ATUALIZOU SOZINHA!! → e gerou etiqueta correios em um teste que fiz!
entregue → aqui sim talvez com o rastreio concluido no status final! delivered
	→ sera que a pluggto atualiza como entregue!?

to achando que nois vai até o faturado!


Camila:

- [x] ver o cancelamento do produto - desativação 🆗
	- [x] Desativação completa do produto → zerar o estoque do produto.
	- [x] Desativação de um marketplace → direto na Pluggto.
- [x] me informar se realmente o dado da fatura que eu envio vai ser com o id do endereço, mas pelo oq eu vi é isso msm!
	- [x] → eu mesmo testar esse caso
- [x] como que eu mando pra aprovar o pedido
	- [x] vai vir pedidor aprovados como pendentes!! vou ter que filtrar os dois na busca dos pedidos. Caso ja esteja aprovado nao faz essa atualização
	→ feito, porém esperando retorno, pois nao estou conseguindo aprovar pedidos pendentes!
		→ deixei meio pronto se for colocar no processar ou no buscar!
- [x] testar o aprovarPedido!
- [x] talvez colocar erro que ta sem preço na hora de processar o produto!!
- [x] ESPERANDO RETORNO DA CAMILA DA APROVACAO DO PEDIDO!

---
Tati:
- [x] ver com a Tati → na busca dos pedidos eu nao to validando busca de pedidos pros marketplaces ativos, ela acha isso interessante? se desativar lá alguns, nao vai atualizar preço/estoque!

colocar vendedor 74 → nos pedidos → OK
payment_method →  na hora de encontrar o depara → OK
LOCAL DOS ITENS NA NOTA → E-COMMERCE → OK
confirmar pedido → quando processar → OK


`<gadget >
  <level id="025" description="Principal">
    <container orientacao="V" tamanhoRelativo="100">
      <grid id="grd_027" entityName="PluggtoPedido" multiplaSelecao="N" useNewGrid="S">
        <title>
          <![CDATA[
          
          
          ACOMPANHAMENTO PLUGG.TO
        
        
        ]]>
        </title>
        <expression type="sql" data-source="MGEDS">
          <![CDATA[
          SELECT 
    PED.PLUGGID, 
    PED.ORIGINALID, 
    PED.STATUS,
    PED.LOG,
    PED.CODCLIENTE,
    PAR.NOMEPARC,
    PED.CODMKTPLC,
    MKT.DESCRMKTPLC,
    PED.NUNOTA,
    PED.NUNOTANFE,
    PED.DHCRIACAO
FROM PLGTPED PED 
INNER JOIN TGFPAR PAR ON PAR.CODPARC = PED.CODCLIENTE
INNER JOIN PLGTMKTPLC MKT ON mkt.codmktplc = PED.CODMKTPLC
WHERE PED.LOG IS NOT NULL
ORDER BY PED.DHCRIACAO
        
        
        ]]>
        </expression>
        <metadata>
          <field name="PLUGGID" label="Plugg ID" type="S" visible="true" useFooter="false"></field>
          <field name="ORIGINALID" label="Original ID" type="S" visible="true" useFooter="false"></field>
          <field name="STATUS" label="Status" type="S" visible="true" useFooter="false"></field>
          <field name="LOG" label="LOG" type="S" visible="true" useFooter="false"></field>
          <field name="CODCLIENTE" label="Cód. Cliente" type="I" visible="true" useFooter="false"></field>
          <field name="NOMEPARC" label="Cliente" type="S" visible="true" useFooter="false"></field>
          <field name="CODMKTPLC" label="Cód. Marketplace" type="I" visible="true" useFooter="false"></field>
          <field name="DESCRMKTPLC" label="Marketplace" type="S" visible="true" useFooter="false"></field>
          <field name="NUNOTA" label="Nro. Único" type="I" visible="true" useFooter="false"></field>
          <field name="NUNOTANFE" label="Nro. Único Pré-Nota" type="I" visible="true" useFooter="false"></field>
          <field name="DHCRIACAO" label="Data Criação Pedido" type="D" visible="true" useFooter="false"></field>
        </metadata>
      </grid>
    </container>
  </level>
</gadget>`