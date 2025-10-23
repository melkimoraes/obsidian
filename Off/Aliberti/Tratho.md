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

Dependentes:

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
            @V_EMAILS    = STRING_AGG(TRIM(USU.EMAIL), ',') 
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
        EXEC sankhya.Stp_Gravafilabi_TRA @V_ASSUNTO, NULL, @V_DATE, 'Pendente', @V_CODCON, 20, @V_HTML, 'E', 3, 'Leandro@aliberti.com.br', @V_CODSMTP, 'text/html';

        FETCH NEXT FROM cur INTO @V_NUREQ, @V_CODFUNC, @V_CODEMP, @V_NOMEDEP, @V_PARENTESCO, @V_NOMEFUNC;
    END

    CLOSE cur;
    DEALLOCATE cur;
END



```


Atestado:
``
```sql
CREATE TRIGGER SANKHYA.TRG_IU_TFPREQAFAS_TRA
ON SANKHYA.TFPREQAFAS
AFTER INSERT, UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE
        @V_HTML          NVARCHAR(MAX),
        @V_NOMEFUNC      NVARCHAR(2000),
        @V_CID           NVARCHAR(2000),
        @V_DTINIOCOR     DATE,
        @V_DTFINALCOR    DATE,
        @V_PERIODO       NVARCHAR(2000),
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
       Objetivo: Envio de e-mail de requisição de afastamento
       *********************************************** */

    -- percorre cada ID afetado (trigger é por instrução no SQL Server)
    DECLARE cur CURSOR LOCAL FAST_FORWARD FOR
        SELECT
            I.ID AS request_id,
            I.CODFUNC,
            I.CODEMP,
            I.NUREINCID AS cid,
            I.DTINIOCOR,
            I.DTFINALCOR,
            FUN.NOMEFUNC AS employee_name
        FROM inserted I
        INNER JOIN SANKHYA.TFPFUN AS FUN ON FUN.CODFUNC = I.CODFUNC AND FUN.CODEMP = I.CODEMP
        WHERE I.ID IS NOT NULL;

    OPEN cur;
    FETCH NEXT FROM cur INTO @V_NUREQ, @V_CODFUNC, @V_CODEMP, @V_CID, @V_DTINIOCOR, @V_DTFINALCOR, @V_NOMEFUNC;

    WHILE @@FETCH_STATUS = 0
    BEGIN
        -- agrega dados do líder e e-mails
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
        WHERE TIPO = 'A'; -- Tipo 'A' para Afastamento

        IF @V_NOMEFUNC IS NULL
        BEGIN
            FETCH NEXT FROM cur INTO @V_NUREQ, @V_CODFUNC, @V_CODEMP, @V_CID, @V_DTINIOCOR, @V_DTFINALCOR, @V_NOMEFUNC;
            CONTINUE;
        END;

        -- Formata o período
        SET @V_PERIODO = CONVERT(VARCHAR(10), @V_DTINIOCOR, 103) + ' a ' + CONVERT(VARCHAR(10), @V_DTFINALCOR, 103);

        -- aplica os placeholders
        SET @V_HTML = REPLACE(@V_HTML, '{{employee_name}}', ISNULL(@V_NOMEFUNC, ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{cid}}', ISNULL(@V_CID, ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{periodo}}', ISNULL(@V_PERIODO, ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{request_id}}', @V_NUREQ);
        SET @V_HTML = REPLACE(@V_HTML, '{{company_name}}', ISNULL(@V_EMPRESA, ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{current_year}}', CONVERT(VARCHAR(4), YEAR(GETDATE())));
        SET @V_HTML = REPLACE(@V_HTML, '{{url_logo}}', ISNULL(@V_URLLOGO, ''));

        SET @V_DATE = GETDATE();

        -- dispara e-mail
        EXEC sankhya.Stp_Gravafilabi_TRA @V_ASSUNTO, NULL, @V_DATE, 'Pendente', @V_CODCON, 20, @V_HTML, 'E', 3, @V_EMAILS, @V_CODSMTP, 'text/html';

        FETCH NEXT FROM cur INTO @V_NUREQ, @V_CODFUNC, @V_CODEMP, @V_CID, @V_DTINIOCOR, @V_DTFINALCOR, @V_NOMEFUNC;
    END

    CLOSE cur;
    DEALLOCATE cur;
END

```

Alteracao Salario
```sql
CREATE TRIGGER SANKHYA.TRG_IU_TFPREQACS_TRA
ON SANKHYA.TFPREQACS
AFTER INSERT, UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE
        @V_HTML          NVARCHAR(MAX),
        @V_NOMEFUNC      NVARCHAR(2000),
        @V_CARGOATUAL    NVARCHAR(2000),
        @V_CARGONOVO     NVARCHAR(2000),
        @V_SALARIOATUAL  DECIMAL(18,2),
        @V_SALARIONOVO   DECIMAL(18,2),
        @V_STATUSLABEL   NVARCHAR(200),
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
       Objetivo: Envio de e-mail de solicitação de alteração de cargo e salário
       *********************************************** */

    -- percorre cada ID afetado (trigger é por instrução no SQL Server)
    DECLARE cur CURSOR LOCAL FAST_FORWARD FOR
        SELECT
            I.ID AS request_id,
            I.CODFUNC,
            I.CODEMP,
            I.CARGO AS cargo_atual_id,
            I.CARGOANTIGO AS cargo_novo_id,
            I.SALARIO AS salario_atual,
            I.SALARIOANTIGO AS salario_novo,
            I.STATUS AS status_code,
            FUN.NOMEFUNC AS employee_name
        FROM inserted I
        INNER JOIN SANKHYA.TFPFUN AS FUN ON FUN.CODFUNC = I.CODFUNC AND FUN.CODEMP = I.CODEMP
        WHERE I.ID IS NOT NULL;

    OPEN cur;
    FETCH NEXT FROM cur INTO @V_NUREQ, @V_CODFUNC, @V_CODEMP, @V_CARGOATUAL, @V_CARGONOVO, @V_SALARIOATUAL, @V_SALARIONOVO, @V_STATUSLABEL, @V_NOMEFUNC;

    WHILE @@FETCH_STATUS = 0
    BEGIN
        -- agrega dados do líder e e-mails
        SELECT
            @V_EMAILS    = STRING_AGG(TRIM(USU.EMAIL), ',') 
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
        WHERE TIPO = 'S'; -- Tipo 'S' para Solicitação de Alteração de Cargo e Salário

        IF @V_NOMEFUNC IS NULL
        BEGIN
            FETCH NEXT FROM cur INTO @V_NUREQ, @V_CODFUNC, @V_CODEMP, @V_CARGOATUAL, @V_CARGONOVO, @V_SALARIOATUAL, @V_SALARIONOVO, @V_STATUSLABEL, @V_NOMEFUNC;
            CONTINUE;
        END;

        -- Busca descrição dos cargos
        SELECT @V_CARGOATUAL = DESCRCARGO FROM SANKHYA.TFPCAR WHERE CODCARGO = @V_CARGOATUAL;
        SELECT @V_CARGONOVO = DESCRCARGO FROM SANKHYA.TFPCAR WHERE CODCARGO = @V_CARGONOVO;

        -- Aplica o F_DESCROPC para status_label
        SET @V_STATUSLABEL = SANKHYA.F_DESCROPC('TFPREQACS','STATUS',@V_STATUSLABEL);

        -- aplica os placeholders
        SET @V_HTML = REPLACE(@V_HTML, '{{employee_name}}', ISNULL(@V_NOMEFUNC, ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{cargo_atual}}', ISNULL(@V_CARGOATUAL, ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{cargo_novo}}', ISNULL(@V_CARGONOVO, ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{salario_atual}}', ISNULL(CONVERT(NVARCHAR, @V_SALARIOATUAL), ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{salario_novo}}', ISNULL(CONVERT(NVARCHAR, @V_SALARIONOVO), ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{status_label}}', ISNULL(@V_STATUSLABEL, ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{request_id}}', @V_NUREQ);
        SET @V_HTML = REPLACE(@V_HTML, '{{company_name}}', ISNULL(@V_EMPRESA, ''));
        SET @V_HTML = REPLACE(@V_HTML, '{{current_year}}', CONVERT(VARCHAR(4), YEAR(GETDATE())));
        SET @V_HTML = REPLACE(@V_HTML, '{{url_logo}}', ISNULL(@V_URLLOGO, ''));

        SET @V_DATE = GETDATE();

        -- dispara e-mail
        EXEC sankhya.Stp_Gravafilabi_TRA @V_ASSUNTO, NULL, @V_DATE, 'Pendente', @V_CODCON, 20, @V_HTML, 'E', 3, 'leandro@aliberti.com.br', @V_CODSMTP, 'text/html';

        FETCH NEXT FROM cur INTO @V_NUREQ, @V_CODFUNC, @V_CODEMP, @V_CARGOATUAL, @V_CARGONOVO, @V_SALARIOATUAL, @V_SALARIONOVO, @V_STATUSLABEL, @V_NOMEFUNC;
    END

    CLOSE cur;
    DEALLOCATE cur;
END

```



```SQL

  -- 2. Busca e-mail do RH (TSIPAR.EMAILRH)
        DECLARE @V_EMAIL_RH NVARCHAR(200);
        SELECT @V_EMAIL_RH = VALOR FROM SANKHYA.TSIPAR WHERE CHAVE = 'EMAILRH';

        -- 3. Concatena e-mails (Líder + RH)
        SET @V_EMAILS = ISNULL(@V_EMAILS_LIDER, '') + 
                        CASE WHEN ISNULL(@V_EMAILS_LIDER, '') <> '' AND ISNULL(@V_EMAIL_RH, '') <> '' THEN ',' ELSE '' END +
                        ISNULL(@V_EMAIL_RH, '');

```