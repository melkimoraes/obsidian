https://crmbonus-api.readme.io/reference/fluxo-da-integracao-pdv

codempresa
e77c7d61-ea31-11ef-808

Athorization
CRM&BonusAPI#2018

BaseURL
https://giftback-sandbox.crmbonus.com/pages

loja 100982
código: 2018

201.71.158.236:8280/mge/

Usuário SUP
Senha adabp2


---

Duvidas a respeito do processo agora:
No Sankhya comentei que faria um botão de ação, que no momento da venda, voces clicariam pra gerar o pin. 
E depois eu faria outro botão que vai perguntar o pin que o cliente te falar. 
Tem problema isso estar separado em 2 botões?

Outra duvida é, o finalizar compra no CRM Bônus exige a NF, então pensando no fluxo e validações.
Só deixaria iniciar o processo no CRM, se já tiver salvado o cabeçalho da nota e também com pelo menos 1 produto inserido na venda, correto?


---

- [x] ver se vai precisar validar TOP pra usar CRM
- [x] ver se no cadastro dos parceiros usa o campo Telefone ou Celular/Fax
- [x] FALAR QUE no final ao dar o bonus, dá sim/nao pro usuario? (eu ja apliquei o desconto)
- [x] ver se a nota deles é o campo STATUSNFE = A
- [x] ver questao do campanha_id → oq passar?
- [x] fazer o validar nota no api inicio
- [ ] criar campos na CAB calculado do CRM!
	- [ ] saldo bonus
	- [ ] campanha
	- [ ] finalizado
	- [ ] cancelado
- [x] valor da consulta bonus → é o valor total dos itens?
- [x] em todas as ações depois do inicio → validar se ainda é do mesmo cliente.
- [x] comentar todos os sysouts
- [x] salvar o celular depois do inicio da api na INTCRM
- [x] salvar o request do finalizar!
- [x] ver questao do question do bonus disponivel → se acumular bonus acredito que o douglas vai querer que apareça!
- [x] fazer o vendas_totais
	- [x] testar
	- [x] e depois tirar os sysouts só dele que faltou!
- [x] colocar filtros na tela Integração CRM (data, parceiro, nro unico)