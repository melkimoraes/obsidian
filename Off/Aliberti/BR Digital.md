```sql
CREATE OR REPLACE PROCEDURE "STP_UPTNATCRITE" (
       P_TIPOEVENTO INT,    -- Identifica o tipo de evento
       P_IDSESSAO VARCHAR2, -- Identificador da execução. Serve para buscar informações dos campos da execução.
       P_CODUSU INT         -- Código do usuário logado
) AS
       BEFORE_INSERT INT;
       AFTER_INSERT  INT;
       BEFORE_DELETE INT;
       AFTER_DELETE  INT;
       BEFORE_UPDATE INT;
       AFTER_UPDATE  INT;
       BEFORE_COMMIT INT;
       V_NUNOTA NUMBER;
       V_SEQUENCIA NUMBER;
       V_COUNT NUMBER;
       V_NUNOTAORIG NUMBER;
       V_SEQUENCIAORIG NUMBER;
       V_CODNAT NUMBER;
       V_CODCENCUS NUMBER;
BEGIN
       BEFORE_INSERT := 0;
       AFTER_INSERT  := 1;
       BEFORE_DELETE := 2;
       AFTER_DELETE  := 3;
       BEFORE_UPDATE := 4;
       AFTER_UPDATE  := 5;
       BEFORE_COMMIT := 10;
       
/*******************************************************************************
Evento para preencher em TOP's parametrizadas, a Natureza e Centro Resultado do pedido de origem nos itens.
********************************************************************************/

IF P_TIPOEVENTO = AFTER_INSERT THEN
        V_NUNOTA := EVP_GET_CAMPO_INT(P_IDSESSAO,  'NUNOTA');
        V_SEQUENCIA := EVP_GET_CAMPO_INT(P_IDSESSAO,  'SEQUENCIA');
        
        BEGIN
        /*SELECT COUNT(1)
        INTO V_COUNT
        FROM TGFCAB
        WHERE TIPMOV='V'
        AND NUNOTA = V_NUNOTA;
        
        IF V_COUNT > 0 THEN*/
            SELECT DISTINCT NUNOTAORIG,SEQUENCIAORIG
            INTO V_NUNOTAORIG, V_SEQUENCIAORIG
            FROM TGFVAR
            WHERE NUNOTA  = V_NUNOTA
            AND SEQUENCIA = V_SEQUENCIA
            AND NUNOTA <> NUNOTAORIG;
        
            SELECT CODNAT, CODCENCUS
            INTO V_CODNAT, V_CODCENCUS
            FROM TGFCAB
            WHERE NUNOTA = V_NUNOTAORIG;
            
            UPDATE TGFITE SET AD_CODNAT=V_CODNAT, AD_CODCENCUS=V_CODCENCUS WHERE NUNOTA = V_NUNOTA AND SEQUENCIA = V_SEQUENCIA;
        EXCEPTION WHEN OTHERS THEN
                  RETURN;
        END;
        --END IF;
    END IF;

END;

```



CabecalhoNota.NUNOTA IN (SELECT NUNOTA FROM TGFITE WHERE AD_CODCENCUS = ?:{entidade=ItemNota;campo=AD_CODCENCUS})

CabecalhoNota.NUNOTA IN (SELECT NUNOTA FROM TGFITE WHERE AD_CODNAT = ?:{entidade=ItemNota;campo=AD_CODNAT})