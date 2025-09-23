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
200.175.146.162:8280/mge/

Usuário SUP
Senha adabp2


---

REENVIAR PIN COM ERRO -> QUANDO TROCA O PARCEIRO DEPOIS DE INICIAR COM OUTRO → OK

65993546319

TRUNCAR PERIODO DA TELA INT CRM! → OK

a tela por empresa → OK
configuração do crm por empresa → OK

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


---

CREATE OR REPLACE TRIGGER TRG_TSILIB_BONUS
BEFORE INSERT ON TSILIB
FOR EACH ROW
DECLARE
    V_USARBONUS   VARCHAR2(1);
    V_SALDOBONUS  NUMBER;
    V_VLRTOTPERC  NUMBER;
BEGIN
    -- Só executa se for EVENTO = 2
    IF :NEW.EVENTO = 2 THEN
        BEGIN
            -- Buscar dados da AD_INTCRM

            SELECT A.USARBONUS, A.SALDOBONUS, C.VLRDESCTOT
              INTO V_USARBONUS, V_SALDOBONUS, V_VLRTOTPERC
              FROM AD_INTCRM A
              JOIN TGFCAB C ON C.NUNOTA = A.NUNOTA
             WHERE A.NUNOTA = :NEW.NUCHAVE;

            -- Validar condições
            IF V_USARBONUS = 'S'
               AND V_SALDOBONUS = V_VLRTOTPERC THEN
                :NEW.VLRLIBERADO := :NEW.VLRATUAL;
                :NEW.DHLIB       := SYSDATE;
            END IF;
        EXCEPTION
            WHEN NO_DATA_FOUND THEN
                RETURN;
        END;
    END IF;
END;