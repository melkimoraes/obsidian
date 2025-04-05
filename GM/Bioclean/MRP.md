- [x] testar exclusao de MRP quando ja tiver!!!
nuMps = 455
codProd = 403


update em banco + commit fora do loop da VWMPS2 -> só insere os produtos e os componentes certos.

update em banco + commit dentro do loop VWMPS2 -> insere os produtos e os componentes e algumas quantidades.

update java dentro do loop VWMPS2, sem commit antes -> buga os produtos e componentes!

update java dentro do loop VWMPS2, com commit antes -> buga os produtos e componentes!

update java fora do loop VWMSP2, com commit antes -> insere os produtos e os componentes e a quantidade certa!

TRG_IU_AD_TPRMRS2_BC  -> deixa muito lento



---

- [x] Ver  o Gerar Compra -> erick ta querendo gerar na vdd uma TOP como se fosse o Orçamento de Venda
- [x] E na dash aonde eles geram as cotaçoes do MRP -> ter opçao de gerar Cotação OU o Pedido de Compra, sem cotar!

Sem. 01 - Qtd. Req. Compra: -> vai ter que olhar a top 1318, nao mais a requisição.

---

https://www.diffchecker.com/Dbmo7yot/