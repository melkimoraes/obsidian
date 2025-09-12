Banco de Dados IMGJ WEVY:
IP: 10.100.114.51
Porta: 1521
Service Name: SANKHYA_PROD
Username: sankhya
Pass:  Uz654EjAaWxG!@

Banco de Dados IMGJ_TESTE (Local)
IP: 172.20.16.119
Porta: 1521
Service Name: IMGJ_TESTE
Username: sankhya
Pass: zsbjVez5JyugGJUI

App SSH:
IP Local: 10.100.114.50:64222
User: imgj_adm
Pass: Imgj@adm

DB SSH:
IP Local: 10.100.114.51:64222
User: imgj_adm
Pass: Imgj@adm

WPM:
IP: http://177.85.34.36:34480/wpm/
DNS: http://erp.imgj.com.br:34480/wpm/
Senha WPM: x1FU!Hg<U53bpEh<

Sankhya OM:
http://erp.imgj.com.br:34480/mge/

SUP:
User: sup
Pass: Uz654EjAaWxG!

Acesso externo: http://177.85.34.36:34480/mge/

(Vou criar um dns ainda)

User: sup

---


--Empresa 1 -> clicar no portal Intercompany -> Transferencia de Papel 
-> vai inserir na IGMJ a TGFCABIMGJ 
-> NUNOTARETART (faturar na Artivinco, vai inserir IXN na IMGJ) 
-> NUNOTARETIMGJ (BOBINAPRODUZIDA = 'S') (INSERT NA TGFCABIMGJPRO) 
->  NUNOTAREMSIMB (artivinco) 
-> NUNOTAENTREMSIMB 
-> NUNOTAREMBOBINA (VER SE APROVOU NA IMGJ!)  (VER SE A NUNOTAREMBOBINA ENTROU NA ARTIVINCO → INSERIR XML?)
-> NUNOTARETBOBINA 
-> NUNOTARETBOBINAIMGJ 
-> NUNOTAPRODUCAO 
-> NUNOTAVENDA (O NUNOTAPEDIMGJ PRECISA ESTAR COM NUMERO DO PEDIDO DA ARTIVINCO E CONFIRMADO PRA GERAR NUNOTAVENDA!) 
-> NUNOTAREMORDEM

---

TRIGGER IXN ARTIVINCO (IMGJ)


DECLARE
V_TIPMOV VARCHAR2(10);
V_NUMNOTA NUMBER;
V_CODPARC NUMBER;
V_VLRNOTA NUMBER;
V_CODEMP NUMBER;
V_CODPARCEMP NUMBER;
V_CODEMPNOTA NUMBER;
V_SERIENOTA VARCHAR2(3);
V_CODTIPOPER NUMBER;
V_DHTIPOPER DATE;
V_NATUREZAOPER VARCHAR2(200);
V_PLACA VARCHAR2(20);
V_CODPARCTRANSP NUMBER;
V_CNPJTRANSP VARCHAR2(14);
V_CNPJPARC VARCHAR2(14);
V_CNPJDEST VARCHAR2(14);
V_CODCFO NUMBER;
V_XNOMEEMIT VARCHAR2(1000);
V_XNOMEDEST VARCHAR2(1000);
V_XNOMETRANSP VARCHAR2(1000);
V_CONTADOR NUMBER;
P_NUARQUIVO NUMBER;
V_CODPARCIMGJ   NUMBER;
V_CHAVENFE TGFNFE.CHAVENFE%TYPE;
V_XMLENVCLI TGFNFE.XMLENVCLI%TYPE;
BEGIN

BEGIN
SELECT CODPARC 
INTO V_CODPARCIMGJ 
FROM TGFCAB
WHERE NUNOTA = 3017606
AND CODPARC = 36781;
EXCEPTION WHEN NO_DATA_FOUND THEN
RETURN;
END; 

    SELECT XMLENVCLI, CHAVENFE
    INTO V_XMLENVCLI,V_CHAVENFE
    FROM TGFNFE
    WHERE NUNOTA = 3017606;


    SELECT CAB.TIPMOV, CAB.NUMNOTA, CAB.CODPARC, CAB.VLRNOTA, CAB.CODEMP, CAB.SERIENOTA, CAB.CODTIPOPER, CAB.DHTIPOPER,  VEI.PLACA, CAB.CODPARCTRANSP
    INTO V_TIPMOV, V_NUMNOTA, V_CODPARC, V_VLRNOTA, V_CODEMPNOTA, V_SERIENOTA, V_CODTIPOPER, V_DHTIPOPER, V_PLACA, V_CODPARCTRANSP
    FROM TGFCAB CAB LEFT JOIN TGFVEI VEI ON (VEI.CODVEICULO = CAB.CODVEICULO)
    WHERE CAB.NUNOTA = 3017606;

    --Empresa do parceiro destinatário
    SELECT MAX(CODEMP)
    INTO V_CODEMP
    FROM TSIEMP@IMGJ
    WHERE CODPARC = V_CODPARC;

    --Parceiro da empresa emitente
    SELECT MAX(CODPARC)
    INTO V_CODPARCEMP
    FROM TSIEMP
    WHERE CODEMP = V_CODEMPNOTA;    

    IF V_TIPMOV = 'V' AND V_CODEMP IS NOT NULL AND V_CODTIPOPER = 1180 THEN

        --Natureza de Operação
        SELECT DESCROPER 
        INTO V_NATUREZAOPER
        FROM TGFTOP@IMGJ 
        WHERE CODTIPOPER = (SELECT TOPRETINDSERV FROM AD_CFGINTEIMGJ@IMGJ WHERE ROWNUM = 1)
        AND DHALTER = (SELECT MAX(DHALTER) FROM TGFTOP@IMGJ WHERE CODTIPOPER =  (SELECT TOPRETINDSERV FROM AD_CFGINTEIMGJ@IMGJ WHERE ROWNUM = 1));

        --Dados da Transportadora
        SELECT MAX(CGC_CPF), MAX(RAZAOSOCIAL)
        INTO V_CNPJTRANSP, V_XNOMETRANSP
        FROM TGFPAR 
        WHERE CODPARC = V_CODPARCTRANSP;

        --Dados do Emitente
        SELECT MAX(CGC_CPF), MAX(RAZAOSOCIAL)
        INTO V_CNPJPARC, V_XNOMEEMIT
        FROM TGFPAR 
        WHERE CODPARC = V_CODPARCEMP;

        --Dados do Destinatário
        SELECT CGC_CPF, RAZAOSOCIAL 
        INTO V_CNPJDEST, V_XNOMEDEST
        FROM TGFPAR 
        WHERE CODPARC = V_CODPARC;

        --CFOP do item
        SELECT MAX(CODCFO)
        INTO V_CODCFO
        FROM TGFITE
        WHERE NUNOTA = 3016884;

            SELECT COUNT(1)
            INTO V_CONTADOR
            FROM TGFIXN@IMGJ
            WHERE CHAVEACESSO =V_CHAVENFE;

            IF V_CONTADOR = 0 THEN

                STP_KEYGEN_TGFNUM@IMGJ(P_ARQUIVO => 'TGFIXN',
                          P_CODEMP  => 1,
                          P_TABELA  => 'TGFIXN',
                          P_CAMPO   => 'NUARQUIVO',
                          P_DSYNC   => 1,
                          p_ultcod  => P_NUARQUIVO);

                INSERT INTO TGFIXN@IMGJ (
                    NUARQUIVO, NOMEARQUIVO, XML, NUMNOTA, CHAVEACESSO, STATUS, DHIMPORT, TIPO, CODUSUIMP, CODEMP, CODPARC, IMPORTADOMDE, SITUACAONFE, SITUACAOMDE,
                    DHEMISS, VLRNOTA, ULTEVEDFE, TOMADORCTE, SERIEDOC, NATUREZAOPER, DTAUTORIZACAO, PLACAVEIC, CNPJTRANSP, CODPARCTRANSP,CNPJPARC,CNPJDEST,
                    ENTSAINFE, CFOPXML, XNOMEEMIT, XNOMEDEST, XNOMETRANSP,CODTIPOPER
                ) VALUES (
                    P_NUARQUIVO,'INSERIDO MANUAL PELA ROTINA IMGJ', V_XMLENVCLI, V_NUMNOTA,V_CHAVENFE,0,SYSDATE,'N',
                    0,V_CODEMP,V_CODPARCEMP,'N',1,4,SYSDATE,V_VLRNOTA,'Ciencia da Operacao','A',V_SERIENOTA,V_NATUREZAOPER, SYSDATE,V_PLACA,V_CNPJTRANSP,V_CODPARCTRANSP,
                    V_CNPJPARC,V_CNPJDEST,1,V_CODCFO,V_XNOMEEMIT,V_XNOMEDEST,V_XNOMETRANSP,(SELECT TOPRETINDSERV FROM AD_CFGINTEIMGJ@IMGJ WHERE ROWNUM = 1)
                );
            END IF;

    END IF;
  
END;

---

Segue acessos da IMGJ:

|                    |                  |                  |
| ------------------ | ---------------- | ---------------- |
| **BANCO DE DADOS** |                  |                  |
|                    | **PRODUCAO**     | **TESTE**        |
| **IP**             | 10.100.114.51    | 172.20.16.119    |
| **PORTA**          | 1521             | 1521             |
| **SERVICE NAME**   | SANKHYA_PROD     | IMGJ_TESTE       |
| **USERNAME**       | sankhya          | sankhya          |
| **PASSWORD**       | !YBSotIsQdlJSc4L | zsbjVez5JyugGJUI |

|              |                  |               |                 |               |
| ------------ | ---------------- | ------------- | --------------- | ------------- |
| **SSH**      |                  |               |                 |               |
|              | **PRODUCAO APP** | **TESTE APP** | **PRODUCAO BD** | **TESTE BD**  |
| **IP**       | 10.100.114.51    | 172.20.16.120 | 10.100.114.50   | 172.20.16.119 |
| **PORTA**    | 64222            | 22            | 64222           | 22            |
| **USERNAME** | imgj_adm         | t495542.adm   | imgj_adm        | t495542.adm   |
| **PASSWORD** | Imgj@adm         | Mudar@123     | Imgj@adm        | Mudar@123     |

---

clico no botão -> é devolução física? gerar nota de remessa industrialização tipmov v da imgj pra artivinco, confirmar.

se precisar tem a proc que o elton já fez algo na artivinco : STP_NEWPED_DEV_ART


````SQL
CREATE OR REPLACE PROCEDURE "STP_PEDDEV_IMGJ" (
       P_CODUSU NUMBER,        -- Código do usuário logado
       P_IDSESSAO VARCHAR2,    -- Identificador da execução. Serve para buscar informações dos parâmetros/campos da execução.
       P_QTDLINHAS NUMBER,     -- Informa a quantidade de registros selecionados no momento da execução.
       P_MENSAGEM OUT VARCHAR2 -- Caso seja passada uma mensagem aqui, ela será exibida como uma informação ao usuário.
) AS
       PARAM_TIPODEV VARCHAR2(4000);
       FIELD_NUNOTA NUMBER;
       V_TIPMOV     VARCHAR2(10);
       V_CONTADOR   NUMBER;
       V_NUNOTADEST NUMBER;
       V_CAB        TGFCAB%ROWTYPE;
       V_DHTIPOPER  DATE;
       V_DHTIPVENDA DATE;
       V_LINK       VARCHAR2(4000);
       V_NUNOTA     NUMBER;
       V_VLRNOTA    NUMBER;
       V_NUMNOTA    NUMBER;
       V_MOD        NUMBER;
       V_RETORNO         VARCHAR (4000);
       
/***********************************************
 Autor:    Elton Aliberti
 Empresa:  Aliberti Consultoria Tecnologica
 Data:     06/05/2025
 Objetivo: Gerar remessa da devolução.
***********************************************/
BEGIN

       PARAM_TIPODEV := ACT_TXT_PARAM(P_IDSESSAO, 'SEMENT');
       
       IF PARAM_TIPODEV = 'FL' THEN
        P_MENSAGEM := 'Ação cancelada. Sem confiugração para devolução fiscal.';
        RETURN;
       END IF;

       FOR I IN 1..P_QTDLINHAS -- Este loop permite obter o valor de campos dos registros envolvidos na execução.
       LOOP                    -- A variável "I" representa o registro corrente.

           FIELD_NUNOTA := ACT_INT_FIELD(P_IDSESSAO, I, 'NUNOTA');

           SELECT TIPMOV
           INTO V_TIPMOV
           FROM TGFCAB
           WHERE NUNOTA = FIELD_NUNOTA;

           IF V_TIPMOV <> 'D' THEN
            STP_MSGERRO('Ação cancelada.', 'Rotina permitida somente para notas de devolução de venda');
           END IF;

           SELECT COUNT(1), MAX(NUNOTA)
           INTO V_CONTADOR, V_NUNOTADEST
           FROM TGFVAR
           WHERE NUNOTAORIG = FIELD_NUNOTA
           AND NUNOTAORIG <> NUNOTA;

           IF V_CONTADOR > 0 THEN
            STP_MSGERRO('Ação cancelada.', 'Já foi gerado o nro. único: '|| V_NUNOTADEST ||' de destino para essa nota de devolução');
           END IF;

           BEGIN
              SELECT NVL(INTEIRO,0)
              INTO V_MOD
              FROM TSIPAR
              WHERE CHAVE = 'MODNFREMDEV';
           EXCEPTION
              WHEN NO_DATA_FOUND THEN
              V_MOD := 0;
           END;

           IF V_MOD = 0 THEN
              STP_MSGERRO('Ação cancelada.', 'O registro selecionado não é um pedido de venda');
           END IF;

           SELECT *
           INTO V_CAB
           FROM TGFCAB
           WHERE NUNOTA = V_MOD;

            SELECT TOP.DHALTER, TOP.TIPMOV
            INTO V_DHTIPOPER, V_TIPMOV
            FROM TGFTOP TOP
            WHERE TOP.CODTIPOPER = V_CAB.CODTIPOPER
            AND TOP.DHALTER = (SELECT MAX(TOP2.DHALTER)
                           FROM TGFTOP TOP2
                           WHERE TOP2.CODTIPOPER = TOP.CODTIPOPER);

            SELECT TPV.DHALTER
            INTO V_DHTIPVENDA
            FROM TGFTPV TPV
            WHERE TPV.CODTIPVENDA = V_CAB.CODTIPVENDA
            AND TPV.DHALTER = (SELECT MAX(TPV2.DHALTER)
                           FROM TGFTPV TPV2
                           WHERE TPV2.CODTIPVENDA = TPV.CODTIPVENDA);

           STP_KEYGEN_TGFNUM (P_ARQUIVO   => 'TGFCAB',
                                P_CODEMP    => V_CAB.CODEMP,
                                P_TABELA    => 'TGFCAB',
                                P_CAMPO     => 'NUNOTA',
                                P_DSYNC     => 1,
                                P_ULTCOD    => V_NUNOTA);

           SELECT SUM(VLRTOT)
           INTO V_VLRNOTA
           FROM TGFITE
           WHERE NUNOTA = FIELD_NUNOTA;

       --     STP_NUMERAR_NOTA2('PEDVEN', V_CAB.CODEMP, ' ', 0, V_CAB.CODTIPOPER, 'P', 'F', V_CAB.CODPARC,' ', V_NUMNOTA);

           INSERT INTO TGFCAB (NUNOTA, NUMNOTA, CODEMP, CODEMPNEGOC, DTNEG, CODUSU, CODUSUINC, DTENTSAI, DTMOV, DTALTER, CODTIPOPER, TIPMOV, DHTIPOPER, CODPARC, CODNAT, CODCENCUS, CODCONTATO, CODCONTATOENTREGA,
                               CODTIPVENDA, DHTIPVENDA, CODVEND, CIF_FOB, CODPARCDEST, VLRNOTA, PESOLIQUIMANUAL, PESOBRUTOMANUAL, PESO, PESOBRUTO)
           VALUES (V_NUNOTA, 0, V_CAB.CODEMP, V_CAB.CODEMP, V_CAB.DTNEG, STP_GET_CODUSULOGADO, STP_GET_CODUSULOGADO, TRUNC(SYSDATE), TRUNC(SYSDATE), SYSDATE, V_CAB.CODTIPOPER, V_TIPMOV, V_DHTIPOPER, V_CAB.CODPARC, V_CAB.CODNAT, 
                               V_CAB.CODCENCUS, V_CAB.CODCONTATO, V_CAB.CODCONTATOENTREGA,
                               V_CAB.CODTIPVENDA, V_DHTIPVENDA, V_CAB.CODVEND, V_CAB.CIF_FOB, V_CAB.CODPARCDEST, V_VLRNOTA, 'S', 'S', V_CAB.PESO, V_CAB.PESOBRUTO);

           INSERT INTO TGFITE (NUNOTA, SEQUENCIA, CODPROD, QTDNEG, VLRUNIT, VLRTOT, CODLOCALORIG, CODVOL, 
                                    NUTAB, CONTROLE, CODEMP, USOPROD, CODCFO, VLRCUS, BASEICMS, VLRICMS, PENDENTE, CODVEND, CODUSU, DTALTER, 
                                    PRECOBASE, NUMPEDIDO2, SEQPEDIDO2, QTDVOL, DTINICIO, ATUALESTOQUE)
                    SELECT V_NUNOTA, SEQUENCIA, CODPROD, QTDNEG, VLRUNIT, VLRTOT, CODLOCALORIG, CODVOL, 
                                    NUTAB, CONTROLE, CODEMP, USOPROD, CODCFO, VLRCUS, BASEICMS, VLRICMS, 'S', CODVEND, STP_GET_CODUSULOGADO, SYSDATE, 
                                    PRECOBASE, NUMPEDIDO2, SEQPEDIDO2, QTDVOL, TRUNC(SYSDATE)+2, 0
                    FROM TGFITE
                    WHERE NUNOTA = FIELD_NUNOTA;

           INSERT INTO TGFVAR (NUNOTA, SEQUENCIA, NUNOTAORIG, SEQUENCIAORIG, STATUSNOTA )
           SELECT V_NUNOTA, SEQUENCIA, V_NUNOTA, SEQUENCIAORIG, STATUSNOTA 
           FROM TGFVAR 
           WHERE NUNOTA = FIELD_NUNOTA
           AND NUNOTA = NUNOTAORIG;

           FOR VR IN (SELECT NUNOTA, SEQUENCIA, QTDNEG
                      FROM TGFITE
                      WHERE NUNOTA = FIELD_NUNOTA)
           LOOP
                INSERT INTO TGFVAR (NUNOTA, SEQUENCIA, NUNOTAORIG, SEQUENCIAORIG, STATUSNOTA, QTDATENDIDA)
                VALUES (V_NUNOTA, VR.SEQUENCIA, VR.NUNOTA, VR.SEQUENCIA, 'L', VR.QTDNEG);
           END LOOP;
           
            STP_CONFNOTA_WS (V_NUNOTA, V_RETORNO);
         
            IF SUBSTR(V_RETORNO, 1, 15) <> 'Nota Confirmada' THEN
            -- Aqui vai o erro
                RAISE_APPLICATION_ERROR(-20101, 'Erro ao Confirmar Remessa - ' || V_RETORNO);
            END IF;

           /*P_MENSAGEM := P_MENSAGEM || '<a href="' || GET_TSIPAR_TEXTO('ENDACESSEXTWGE') || '/system.jsp#app/'
              || UTL_RAW.cast_to_varchar2(UTL_ENCODE.base64_encode(UTL_RAW.cast_to_raw('br.com.sankhya.com.mov.CentralNotas')))
              || '/'
              || UTL_RAW.cast_to_varchar2(UTL_ENCODE.base64_encode(UTL_RAW.cast_to_raw('{"NUNOTA":'
                                                                                       || TO_CHAR(V_NUNOTA)
                                                                                       || '}')))
              || '" target="_parent"><br/><u>'|| V_NUNOTA ||'</u>';   */
              
            P_MENSAGEM := P_MENSAGEM || '<br/><u>'|| V_NUNOTA ||'</u>';

       END LOOP;

       P_MENSAGEM := 'Remessa(s) gerada(s) com sucesso. Acesse: ' || P_MENSAGEM;END;
/
```

