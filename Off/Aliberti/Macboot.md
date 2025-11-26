Tela de Pedido de Venda

Romaneio  → multiplos: 8, 10, 12, 15

Cabeçalho
Itens → aqui vai ser o produto generico

Gerar pedido de venda/nota de venda → aqui vai inserir os itens com base no EAN rateado pela numeração!

- [x] inserção dos itens vai ser conforme multiplos ou qtd normal?
- [x] pedido de venda ou venda vai inserir?
- [x] campo calculado statusnota da cab, nunota da cab
- [x] campo vinculado o nunota na TGFCAB → AD_NUNOTAPED
- [x] EVENTOS
- [x] PREFERENCIA! CODLOCALPEDVEND
- [x] ao atualizar a AD_TGFITE → atualiza o VLRTOT tb!
- [x] GET_LINK_TELA(‘br.com.sankhya.com.mov.CentralNotas‘, ‘Clique aqui para visualizar o Pedido‘, ‘{NUNOTA : 3}‘) - colocar clique aqui pra abrir o pedido?

---

- [x] Verificar a possibilidade de trazer somente os números cadastrados 
- [x] Precisa puxar o preço do produto automático → ?
- [x] Desconto pode ser até 5%, passando disso, a diferença abate da comissão.
- [x] Verificar a impressão do pedido→ ?
- [x] Tipo de negociação padrão pra cada cliente ( Criar tabelinha de tipo de negociação no cadastro do parceiro)
	 
---

- [ ] ver o evento da AD_TGFITENUM e criar dnv na macboot!


---


Botão de ação na composição → vai criar outra composição do mesmo produto porem por numeração do sapato! 3,33% pra cada numeração. Só vai fazer esse calculo em cima da atividade de CORTE.
pra cima é 3,33% pra cima
pra menos é 3,33% pra menos

sempre vai aplicar da base que é o tamanho 39 → pro tamanho dele!
vai ter um campo lista do padrao (por ta trabalhando com controle de lista!) NA TPRLPA

só aplicar na TPRMPA

---

Relatórios:

Ficha de produção
- NUMPS na TPRIPRO

Agrupamento
- ficha é os números da OP

---

SELECT 
MPS.NUMPS,
MPS.DHGERMRP,
PRO.REFERENCIA,
PRO.DESCRPROD,
PRO.COMPLDESC,
PRO.AD_FORMA,
PRO.AD_MENOR,
PRO.AD_MAIOR,
PRO.AD_BASE,
to_char(sysdate, 'DD/MM/YYYY HH24:MI:SS') as dt,
to_char(PRO.DTALTER, 'DD/MM/YYYY HH24:MI:SS') as dtalter,
PRO.IMAGEM
FROM TPRMPS MPS
INNER JOIN TGFPRO PRO ON PRO.CODPROD = (SELECT CODPROD FROM TPRIMPS WHERE NUMPS = MPS.NUMPS AND ROWNUM=1)
WHERE MPS.NUMPS = 6

FICHA : ORDEM DE PRODUÇÃO -> TEM MAIS DE UMA ORDEM DE PRODUÇÃO NO MSM NUMPS.

PEDIDO A MESMA COISA E CLIENTE A MESMA COISA.

O PRODUTO QUE APARECE, DESCRICAO E ETC -> PEGO QUAL PRODUTO DA TPRMPS.

OU É PRA AGRUPAR POR PEDIDO? 

---

- Aparecer só as numerações do produto:
	- quando inserir o produto ja inserir as numerações automaticamente zeradas
- Desconto: criar campo com percentual de desconto que o vendedor vai querer colocar e replicar isso nos itens do pedido.
- ireport do pedido(ta incluso?)
- Criar aba do tipo de negociação que o cliente vai poder usar.

---

Ficha produção do corte do couro -> é por OP? faz igual do exemplo? a aparte do meio toda é informações fixas para serem preenchidas? 

Ficha de preparação do forro -> mesma duvida da produção do corte do couro!

ficha agrupada preparação - é por OP? por MPS? é pra somar as MP de cada OP dai?

EVA -> vai imprimir da OP tb? aonde tem os produtos listados no exemplo? é alguma atividade da OP?

---

ORDEM DE PRODUÇÃO DO CORTE DO COURO -> pegar os produtos do grupo 40202000 pra exibir como MP → ok

FICHA DE PREPARAÇÃO DO FORRO -> todos os produto da LMP -> corte também → ok

FICHA AGRUPADA PREPARAÇÃO -> todos produtos da LMP -> atividade preparação. → ok

PRODUCAO EVA -> PRODUTOS SERAO LMP DO PRODUTOPA → ok

---

Acessos

●
 Produção- macboot.sankhyacloud.com.br
●
 Teste - macboot-teste.sankhyacloud.com.br
●
 Aplicativos - app.macboot.sankhyacloud.com.br
●
 E-social - esocial-macboot.sankhyacloud.com.br/index.html



 Senha do usuário SUP :


●
 Produção - 0VG42OPm4qoDGf3mp6
●
 Teste - 0VG42OPm4qoDGf3mp6
 Senha do WPM
●
 Produção - 6f7bZ7ZowEno
●
 Teste - 6f7bZ7ZowEno