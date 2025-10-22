Acessos
Acessos

https://sankhyaom1.tratho.com.br/mge/

SUP
``Tratho$$2025``


FWC

https://suportesankhya.fwc.cloud/
Usuário: sankhya118.suporte
senha: Gte7*23De2

----------------------------------------------------------


```sql

CREATE TRIGGER SANKHYA.TRG_IU_TFPREQDPD_TRA
ON SANKHYA.TFPREQDPD
AFTER INSERT, UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE
        @V_HTML          NVARCHAR(MAX),
        @V_NOMEFUNC      NVARCHAR(2000),
        @V_NOMEDEP       NVARCHAR(2000),
        @V_PARENTESCO    NVARCHAR(2000),
        @V_EMAILS        NVARCHAR(4000),
        @V_ASSUNTO       NVARCHAR(4000),
        @V_CODFUNC       INT,
        @V_CODEMP        INT,
        @V_CODCON        INT,
        @V_DATE          DATE,
        @V_CODSMTP       INT,
        @V_NUREQ         INT,
        @V_URLLOGO       VARCHAR(2000),
        @V_EMPRESA       NVARCHAR(2000);

    /* ***********************************************
       Empresa:  Aliberti Consultoria Tecnologica
       Objetivo: Envio de e-mail de requisição de dependente
       *********************************************** */

    -- percorre cada NUREQ afetado (trigger é por instrução no SQL Server)
    DECLARE cur CURSOR LOCAL FAST_FORWARD FOR
        SELECT
            I.ID AS request_id,
            FUN.CODFUNC,
            FUN.CODEMP,
            I.NOME AS dependent_name,
            SANKHYA.F_DESCROPC('TFCDEP','GRAUPARENTESCO',I.GRAUPARENT) AS relationship,
            FUN.NOMEFUNC AS employee_name
        FROM inserted I
        INNER JOIN SANKHYA.TFPFUN AS FUN ON FUN.CODFUNC = I.CODFUNC AND FUN.CODEMP = I.CODEMP
        WHERE I.ID IS NOT NULL;

    OPEN cur;
    FETCH NEXT FROM cur INTO @V_NUREQ, @V_CODFUNC, @V_CODEMP, @V_NOMEDEP, @V_PARENTESCO, @V_NOMEFUNC;

    WHILE @@FETCH_STATUS = 0
    BEGIN
        -- agrega dados do funcionário e e-mails do RH (ou líder, se a lógica for mantida)
        SELECT
            @V_EMAILS    = STRING_AGG(TRIM(USU.EMAIL), ",") 
                           WITHIN GROUP (ORDER BY USU.EMAIL)
        FROM SANKHYA.TFPLIDER AS LID
        INNER JOIN SANKHYA.TFPFUN  AS LIR
            ON LIR.CODEMP  = LID.CODEMPLIDER
           AND LIR.CODFUNC = LID.CODFUNCLIDER
        INNER JOIN SANKHYA.TFPFUN  AS FUN
            ON FUN.CODEMP  = LID.CODEMP
           AND FUN.CODFUNC = LID.CODFUNC
        INNER JOIN SANKHYA.TSIUSU  AS USU
            ON USU.CODEMP  = LIR.CODEMP
           AND USU.CODFUNC = LIR.CODFUNC
        WHERE LID.CODFUNC = @V_CODFUNC
        AND LID.CODEMP = @V_CODEMP;

        -- busca template HTML
        SELECT @V_HTML = CONTEUDO,
               @V_ASSUNTO = ASSUNTO,
               @V_CODSMTP = CODSMTP,
               @V_URLLOGO = URLLOGO,
               @V_EMPRESA = EMPRESA
        FROM SANKHYA.AD_CFGEMAIL
        WHERE TIPO = 'D'; -- Tipo 'D' para Dependente

        IF @V_NOMEFUNC IS NULL
        BEGIN
            FETCH NEXT FROM cur INTO @V_NUREQ, @V_CODFUNC, @V_CODEMP, @V_NOMEDEP, @V_PARENTESCO, @V_NOMEFUNC;
            CONTINUE;
        END;

        -- aplica os placeholders
        SET @V_HTML = REPLACE(@V_HTML, '{{employee_name}}', ISNULL(@V_NOMEFUNC, ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{dependent_name}}', ISNULL(@V_NOMEDEP, ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{relationship}}', ISNULL(@V_PARENTESCO, ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{request_id}}', @V_NUREQ);
        SET @V_HTML = REPLACE(@V_HTML, '{{company_name}}', ISNULL(@V_EMPRESA, ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{current_year}}', CONVERT(VARCHAR(4), YEAR(GETDATE())));
        SET @V_HTML = REPLACE(@V_HTML, '{{url_logo}}', ISNULL(@V_URLLOGO, ''));

        SET @V_DATE = GETDATE();

        -- dispara e-mail
        EXEC sankhya.Stp_Gravafilabi_TRA @V_ASSUNTO, NULL, @V_DATE, 'Pendente', @V_CODCON, 20, @V_HTML, 'E', 3, @V_EMAILS, @V_CODSMTP, 'text/html';

        FETCH NEXT FROM cur INTO @V_NUREQ, @V_CODFUNC, @V_CODEMP, @V_NOMEDEP, @V_PARENTESCO, @V_NOMEFUNC;
    END

    CLOSE cur;
    DEALLOCATE cur;
END

```


