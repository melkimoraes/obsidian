http://protec.sankhyacloud.com.br
sup
9h2-@RDLp\Zt5_&*JW

```SQL
WITH PRECOS_VIGENTES AS (
  SELECT
    EXC.CODPROD,
    EXC.VLRVENDA,
    TAB.DTVIGOR,
    ROW_NUMBER() OVER(PARTITION BY EXC.CODPROD ORDER BY TAB.DTVIGOR DESC) as RN
  FROM TGFEXC EXC
  INNER JOIN TGFTAB TAB ON EXC.NUTAB = TAB.NUTAB
  WHERE TAB.CODTAB = 0
)
SELECT 
  PRO.CODPROD,
  PV.VLRVENDA AS ULTIMO_VLRVENDA,
  OBTEMCUSTO(PRO.CODPROD, 'S', 1, 'N', NULL, 'N', NULL, SYSDATE, 3) AS ULTCUS,
  CASE :V_CODEMP
    WHEN 1 THEN '11%' -- Protec Custo Fixo
    WHEN 2 THEN '7%'  -- Tecmar Custo Fixo
    ELSE 'N/A'
  END AS CUSTO_FIXO_PERC,
  CASE :V_CODEMP
    WHEN 1 THEN '9%' -- Protec Custo Variável
    WHEN 2 THEN '6%'  -- Tecmar Custo Variável
    ELSE 'N/A'
  END AS CUSTO_VARIAVEL_PERC,
  CASE :V_CODEMP
    WHEN 1 THEN '5%' -- Protec Margem de Lucro
    WHEN 2 THEN '10%' -- Tecmar Margem de Lucro
    ELSE 'N/A'
  END AS MARGEM_LUCRO_PERC,
  CASE :V_CODEMP
    WHEN 1 THEN 
      OBTEMCUSTO(PRO.CODPROD, 'S', 1, 'N', NULL, 'N', NULL, SYSDATE, 3) * (1 + 0.11 + 0.09 + 0.05)
    WHEN 2 THEN 
      OBTEMCUSTO(PRO.CODPROD, 'S', 1, 'N', NULL, 'N', NULL, SYSDATE, 3) * (1 + 0.07 + 0.06 + 0.10)
    ELSE 0
  END AS NOVO_VLRVENDA
FROM TGFPRO PRO
INNER JOIN PRECOS_VIGENTES PV ON PRO.CODPROD = PV.CODPROD
WHERE PV.RN = 1;


```


```SQL
CREATE OR REPLACE FUNCTION OBTEMCUSTO_PRT (
  P_CODPROD IN NUMBER,
  P_POREMP IN CHAR,
  P_CODEMP IN NUMBER,
  P_PORLOCAL IN CHAR,
  P_CODLOCAL IN NUMBER,
  P_PORCONTROLE IN CHAR,
  P_CONTROLE IN VARCHAR2,
  P_DATA IN DATE,
  P_TIPO IN NUMBER)
RETURN FLOAT
IS  P_CUSREP          TGFCUS.CUSREP%TYPE;
    P_CUSGER          TGFCUS.CUSGER%TYPE;
    P_CUSVARIAVEL     TGFCUS.CUSVARIAVEL%TYPE;
    P_CUSSEMICM       TGFCUS.CUSSEMICM%TYPE;
    P_CUSMEDICM       TGFCUS.CUSMEDICM%TYPE;
    P_ENTRADASEMICMS  TGFCUS.ENTRADASEMICMS%TYPE;
    P_ENTRADACOMICMS  TGFCUS.ENTRADACOMICMS%TYPE;
    P_CUSMED          TGFCUS.CUSMED%TYPE;
BEGIN
  BEGIN  
    SELECT NVL(MAX(CUSREP),0), 
           NVL(MAX(CUSGER),0), 
           NVL(MAX(CUSVARIAVEL),0), 
           NVL(MAX(CUSSEMICM),0), 
           NVL(MAX(CUSMEDICM),0), 
           NVL(MAX(ENTRADASEMICMS),0), 
           NVL(MAX(ENTRADACOMICMS),0),
           NVL(MAX(CUSMED),0)
      INTO P_CUSREP, 
           P_CUSGER, 
           P_CUSVARIAVEL, 
           P_CUSSEMICM, 
           P_CUSMEDICM, 
           P_ENTRADASEMICMS, 
           P_ENTRADACOMICMS,
           P_CUSMED
      FROM TGFCUS
     WHERE CODPROD = P_CODPROD
       AND (P_PORCONTROLE  = 'N' OR CONTROLE = P_CONTROLE)
       AND (P_PORLOCAL = 'N' OR CODLOCAL = P_CODLOCAL)
       AND (P_POREMP = 'N' OR CODEMP = P_CODEMP)
       AND DTATUAL = (SELECT MAX(DTATUAL) FROM TGFCUS CN
                       WHERE CODPROD = P_CODPROD
                         AND DTATUAL <= P_DATA
                         AND (P_PORCONTROLE  = 'N' OR CONTROLE = P_CONTROLE)
                         AND (P_PORLOCAL = 'N' OR CODLOCAL = P_CODLOCAL)
                         AND (P_POREMP = 'N' OR CODEMP = P_CODEMP)
                     );
  EXCEPTION
  WHEN NO_DATA_FOUND THEN 
    RETURN 0;
  WHEN OTHERS THEN
    RETURN 0;
  END;

  IF P_TIPO = 0 THEN
    RETURN(P_CUSREP);
  ELSIF P_TIPO = 1 THEN
    RETURN(P_CUSGER);
  ELSIF P_TIPO = 2 THEN
    RETURN(P_CUSVARIAVEL);
  ELSIF P_TIPO = 3 THEN
    RETURN(P_CUSSEMICM);
  ELSIF P_TIPO = 4 THEN
    RETURN(P_CUSMEDICM);
  ELSIF P_TIPO = 5 THEN
    RETURN(P_ENTRADASEMICMS);
  ELSIF P_TIPO = 6 THEN
    RETURN(P_ENTRADACOMICMS);
  ELSIF P_TIPO = 7 THEN
    RETURN(P_CUSMED);  
  ELSE
    RETURN (0);
  END IF;
EXCEPTION
   WHEN NO_DATA_FOUND THEN RETURN (0);
END;

```



```SQL
CREATE OR REPLACE PROCEDURE "STP_CARPRODPRC_PTC" (

       P_CODUSU NUMBER,        -- Código do usuário logado

       P_IDSESSAO VARCHAR2,    -- Identificador da execução. Serve para buscar informações dos parâmetros/campos da execução.

       P_QTDLINHAS NUMBER,     -- Informa a quantidade de registros selecionados no momento da execução.

       P_MENSAGEM OUT VARCHAR2 -- Caso seja passada uma mensagem aqui, ela será exibida como uma informação ao usuário.

) AS

       PARAM_CODEMP VARCHAR2(4000);

       FIELD_ID NUMBER;

BEGIN

  

       PARAM_CODEMP := ACT_TXT_PARAM(P_IDSESSAO, 'CODEMP');

  

       FOR I IN 1..P_QTDLINHAS -- Este loop permite obter o valor de campos dos registros envolvidos na execução.

       LOOP                    -- A variável "I" representa o registro corrente.

  

              SELECT NVL(MAX(ID),0)+1

              INTO FIELD_ID

              FROM AD_ATTPRC ;

  

              INSERT INTO AD_ATTPRC (ID,DTINC,CODEMP,STATUS) VALUES (FIELD_ID,SYSDATE,PARAM_CODEMP,'P');

  

              FOR X IN (WITH PRECOS_VIGENTES AS (

                     SELECT

                     EXC.CODPROD,

                     EXC.VLRVENDA,

                     TAB.DTVIGOR,

                     ROW_NUMBER() OVER(PARTITION BY EXC.CODPROD ORDER BY TAB.DTVIGOR DESC) as RN

                     FROM TGFEXC EXC

                     INNER JOIN TGFTAB TAB ON EXC.NUTAB = TAB.NUTAB

                     WHERE TAB.CODTAB = 0

                     )

                     SELECT

                     PRO.CODPROD,

                     PV.VLRVENDA AS ULTIMO_VLRVENDA,

                     OBTEMCUSTO(PRO.CODPROD, 'S', 1, 'N', NULL, 'N', NULL, SYSDATE, 3) AS ULTCUS,

                     CASE PARAM_CODEMP

                     WHEN 1 THEN '11%' -- Protec Custo Fixo

                     WHEN 2 THEN '7%'  -- Tecmar Custo Fixo

                     ELSE 'N/A'

                     END AS CUSTO_FIXO_PERC,

                     CASE PARAM_CODEMP

                     WHEN 1 THEN '9%' -- Protec Custo Variável

                     WHEN 2 THEN '6%'  -- Tecmar Custo Variável

                     ELSE 'N/A'

                     END AS CUSTO_VARIAVEL_PERC,

                     CASE PARAM_CODEMP

                     WHEN 1 THEN '5%' -- Protec Margem de Lucro

                     WHEN 2 THEN '10%' -- Tecmar Margem de Lucro

                     ELSE 'N/A'

                     END AS MARGEM_LUCRO_PERC,

                     CASE PARAM_CODEMP

                     WHEN 1 THEN

                     OBTEMCUSTO(PRO.CODPROD, 'S', 1, 'N', NULL, 'N', NULL, SYSDATE, 3) * (1 + 0.11 + 0.09 + 0.05)

                     WHEN 2 THEN

                     OBTEMCUSTO(PRO.CODPROD, 'S', 1, 'N', NULL, 'N', NULL, SYSDATE, 3) * (1 + 0.07 + 0.06 + 0.10)

                     ELSE 0

                     END AS NOVO_VLRVENDA

                     FROM TGFPRO PRO

                     INNER JOIN PRECOS_VIGENTES PV ON PRO.CODPROD = PV.CODPROD

                     WHERE PV.RN = 1

                     AND PRO.CODPROD NOT IN (221)) LOOP

                            INSERT INTO AD_ATTPRCPROD (CODPROD, ID, VLRVENDAATUAL, ULTCUS, PERCFIXO, PERCVARIAVEL, MARGEMLUCRO, VLRVENDAATT)

                            VALUES (X.CODPROD, FIELD_ID, X.ULTIMO_VLRVENDA, X.ULTCUS, X.CUSTO_FIXO_PERC, X.CUSTO_VARIAVEL_PERC, X.MARGEM_LUCRO_PERC, X.NOVO_VLRVENDA);

              END LOOP;

  

       END LOOP;

       P_MENSAGEM := 'Gerado Atualizador com Sucesso!';

END;
```