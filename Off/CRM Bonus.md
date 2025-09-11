https://crmbonus-api.readme.io/reference/fluxo-da-integracao-pdv

codempresa
e77c7d61-ea31-11ef-808

Athorization
CRM&BonusAPI#2018

BaseURL
https://giftback-sandbox.crmbonus.com/pages

loja 100982


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
- [ ] FALAR QUE no final ao dar o bonus, dá sim/nao pro usuario? (eu ja apliquei o desconto)
- [x] ver se a nota deles é o campo STATUSNFE = A
- [x] ver questao do campanha_id → oq passar?
- [x] fazer o validar nota no api inicio
- [ ] criar campos na CAB calculado do CRM!
- [x] valor da consulta bonus → é o valor total dos itens?
- [x] em todas as ações depois do inicio → validar se ainda é do mesmo cliente.
- [ ] comentar todos os sysouts
- [x] salvar o celular depois do inicio da api na INTCRM
- [x] salvar o request do finalizar!
- [ ] ver questao do question do bonus disponivel que o douglas vai responder, ocmo também aquela mensagem de assinar algo.
- [ ] ver o vendas_totais?