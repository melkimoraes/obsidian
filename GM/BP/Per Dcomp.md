````
<gadget >
  <prompt-parameters>
    <parameter  id="CODEMP" description="Empresa" metadata="entity:Empresa@CODEMP" required="true" keep-last="true" keep-date="false" label="CODEMP : Entidade/Tabela" order="0"/>
    <parameter  id="NROPERDCOMP" description="Nro. da Per/Dcomp" metadata="text" required="false" keep-last="false" keep-date="false" label="NROPERDCOMP : Texto" order="1"/>
    <parameter  id="TIPCRE" description="Tipo de Crédito" metadata="entity:PRDCMPTIPCRE@ID" required="false" keep-last="true" keep-date="false" label="TIPCRE : Entidade/Tabela" order="2"/>
    <parameter  id="DTPED" description="Periodo dos Lançamentos dos Pedidos" metadata="datePeriod" required="false" keep-last="true" keep-date="false" label="DTPED : Período" order="3"/>
  </prompt-parameters>
  <level id="0FG" description="Principal">
    <args>
      <arg  id="A_SEQ" type="integer" label="A_SEQ : Número Inteiro"/>
    </args>
    <container orientacao="V" tamanhoRelativo="100">
      <container orientacao="V" tamanhoRelativo="50">
        <grid id="grd_0FH" useNewGrid="S">
          <expression type="sql" data-source="MGEDS">
            <![CDATA[
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
          
          
          SELECT SEQ,NROPERDCOMP,
(SELECT
SUM(VLRLANC) AS VLRCOMP
FROM PRDCMPLANCPED  WHERE SEQ=PRDCMPCTR.SEQ AND TIPDOC IN (SELECT ID FROM PRDCMPTIPDOC WHERE TIPO = 'CO')) AS VLRCOMP,
(SELECT
SUM(VLRLANC) AS VLRCOMP
FROM PRDCMPLANCPED  WHERE SEQ=PRDCMPCTR.SEQ AND TIPDOC IN (SELECT ID FROM PRDCMPTIPDOC WHERE TIPO = 'CR')) AS VLRCRED,
(SELECT 
SUM(VLRLANC) AS VLRCRED
FROM PRDCMPLANCPED  WHERE SEQ=PRDCMPCTR.SEQ AND TIPDOC IN (SELECT ID FROM PRDCMPTIPDOC WHERE TIPO = 'CR'))
-
(SELECT
SUM(VLRLANC) AS VLRCOMP
FROM PRDCMPLANCPED  WHERE SEQ=PRDCMPCTR.SEQ AND TIPDOC IN (SELECT ID FROM PRDCMPTIPDOC WHERE TIPO = 'CO')) AS SALDOACOMP,
(SELECT CODEMP || ' - ' || NOMEFANTASIA FROM TSIEMP WHERE CODEMP = PRDCMPCTR.CODEMP) AS CODEMP
FROM PRDCMPCTR
WHERE CODEMP = :CODEMP
AND (CASE WHEN :NROPERDCOMP IS NULL THEN '1' ELSE NROPERDCOMP END) =  (CASE WHEN :NROPERDCOMP IS NULL THEN '1' ELSE :NROPERDCOMP END)
        
        
        
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          
          ]]>
          </expression>
          <metadata>
            <field name="SEQ" label="Sequencia" type="I" visible="true" useFooter="false"></field>
            <field name="NROPERDCOMP" label="Nro. da Per/Dcomp" type="S" visible="true" useFooter="false"></field>
            <field name="VLRCOMP" label="Valor total das compensações" type="F" visible="true" useFooter="false"></field>
            <field name="VLRCRED" label="Valor total dos créditos" type="F" visible="true" useFooter="false"></field>
            <field name="SALDOACOMP" label="Saldo a compensar" type="F" visible="true" useFooter="false"></field>
            <field name="CODEMP" label="Empresa" type="S" visible="true" useFooter="false"></field>
          </metadata>
          <refresh-details  ui-list="grd_0GD,grd_0GF">
            <param id="A_SEQ">$SEQ</param>
          </refresh-details>
        </grid>
      </container>
      <container orientacao="H" tamanhoRelativo="50">
        <container orientacao="H" tamanhoRelativo="50">
          <grid id="grd_0GD" useNewGrid="S">
            <args>
              <arg id="A_SEQ" type="int"></arg>
            </args>
            <expression type="sql" data-source="MGEDS">
              <![CDATA[
              SELECT 
    LANC.SEQ, 
    LANC.SEQLANC, 
    LANC.DATA, 
    LANC.TIPDOC || ' - ' || DOC.DESCDOC AS TIPDOC, 
    LANC.NROPERDCOMP,
    LANC.TIPCRE || ' - ' || CRE.DESCCRED AS TIPCRE, 
    (SELECT CODCRE || ' - ' || DENOMINACAO FROM PRDCMPCODCRE WHERE CODCRE = LANC.CODCRED) AS CODCRED, 
    (SELECT CODCRE || ' - ' || DENOMINACAO FROM PRDCMPCODCRE WHERE CODCRE = LANC.CODCREDDARF) AS CODCREDDARF,
    LANC.DTINC, 
    (SELECT CODUSU || ' - ' || NOMEUSU FROM TSIUSU WHERE CODUSU = LANC.CODUSUINC) AS CODUSUINC, 
    LANC.DTALT, 
    (SELECT CODUSU || ' - ' || NOMEUSU FROM TSIUSU WHERE CODUSU = LANC.CODUSUALT) AS CODUSUALT, 
    LANC.VLRLANC 
FROM PRDCMPLANCPED LANC
INNER JOIN PRDCMPTIPDOC DOC ON DOC.ID = LANC.TIPDOC
INNER JOIN PRDCMPTIPCRE CRE ON CRE.ID = LANC.TIPCRE
WHERE LANC.SEQ = 1 AND DOC.TIPO = 'CR' 
AND (CRE.ID = :TIPCRE OR :TIPCRE IS NULL)
AND (LANC.DATA BETWEEN :DTPED.INI AND :DTPED.FIN OR (:DTPED.INI IS NULL AND :DTPED.FIN IS NULL))
            ]]>
            </expression>
            <metadata>
              <field name="SEQ" label="Nro. Sequencial" type="I" visible="true" useFooter="false"></field>
              <field name="SEQLANC" label="Nro. Sequencial Lanc." type="I" visible="true" useFooter="false"></field>
              <field name="DATA" label="Data" type="D" visible="true" useFooter="false"></field>
              <field name="TIPDOC" label="Tipo de documento" type="S" visible="true" useFooter="false"></field>
              <field name="NROPERDCOMP" label="Nº da PER/DCOMP" type="S" visible="true" useFooter="false"></field>
              <field name="TIPCRE" label="Tipo de Crédito" type="S" visible="true" useFooter="false"></field>
              <field name="CODCRED" label="Cód. do Crédito" type="I" visible="true" useFooter="false"></field>
              <field name="CODCREDDARF" label="Cód. do Crédito DARF" type="I" visible="true" useFooter="false"></field>
              <field name="DTINC" label="Data/Hora de inclusão" type="T" visible="true" useFooter="false"></field>
              <field name="CODUSUINC" label="Usuário de inclusão" type="S" visible="true" useFooter="false"></field>
              <field name="DTALT" label="Data/Hora da últ. alteração" type="T" visible="true" useFooter="false"></field>
              <field name="CODUSUALT" label="Usuário da últ. alteração" type="S" visible="true" useFooter="false"></field>
              <field name="VLRLANC" label="Valor do Lançamento" type="F" visible="true" useFooter="SUM"></field>
            </metadata>
          </grid>
        </container>
        <container orientacao="H" tamanhoRelativo="50">
          <grid id="grd_0GF" useNewGrid="S">
            <args>
              <arg id="A_SEQ" type="Número Inteiro"></arg>
            </args>
            <expression type="sql" data-source="MGEDS">
              <![CDATA[SELECT 
    LANC.SEQ, 
    LANC.SEQLANC, 
    LANC.DATA, 
    LANC.TIPDOC || ' - ' || DOC.DESCDOC AS TIPDOC, 
    LANC.NROPERDCOMP, 
    LANC.TIPCRE || ' - ' || CRE.DESCCRED AS TIPCRE, 
    (SELECT CODCRE || ' - ' || DENOMINACAO FROM PRDCMPCODCRE WHERE CODCRE = LANC.CODCRED) AS CODCRED, 
    (SELECT CODCRE || ' - ' || DENOMINACAO FROM PRDCMPCODCRE WHERE CODCRE = LANC.CODCREDDARF) AS CODCREDDARF,
    LANC.DTINC, 
    (SELECT CODUSU || ' - ' || NOMEUSU FROM TSIUSU WHERE CODUSU = LANC.CODUSUINC) AS CODUSUINC, 
    LANC.DTALT, 
    (SELECT CODUSU || ' - ' || NOMEUSU FROM TSIUSU WHERE CODUSU = LANC.CODUSUALT) AS CODUSUALT, 
    LANC.VLRLANC 
FROM PRDCMPLANCPED LANC
INNER JOIN PRDCMPTIPDOC DOC ON DOC.ID = LANC.TIPDOC
INNER JOIN PRDCMPTIPCRE CRE ON CRE.ID = LANC.TIPCRE
WHERE LANC.SEQ = :A_SEQ AND DOC.TIPO = 'CO'     
AND (CRE.ID = :TIPCRE OR :TIPCRE IS NULL)
AND (LANC.DATA BETWEEN :DTPED.INI AND :DTPED.FIN OR (:DTPED.INI IS NULL AND :DTPED.FIN IS NULL))]]>
            </expression>
            <metadata>
              <field name="SEQ" label="Nro. Sequencial" type="I" visible="true" useFooter="false"></field>
              <field name="SEQLANC" label="Nro. Sequencial Lanc." type="I" visible="true" useFooter="false"></field>
              <field name="DATA" label="Data" type="D" visible="true" useFooter="false"></field>
              <field name="TIPDOC" label="Tipo de documento" type="S" visible="true" useFooter="false"></field>
              <field name="NROPERDCOMP" label="Nº da PER/DCOMP" type="S" visible="true" useFooter="false"></field>
              <field name="TIPCRE" label="Tipo do Crédito" type="S" visible="true" useFooter="false"></field>
              <field name="CODCRED" label="Cód. do Crédito" type="I" visible="true" useFooter="false"></field>
              <field name="CODCREDDARF" label="Cód. do Crédito DARF" type="I" visible="true" useFooter="false"></field>
              <field name="DTINC" label="Data/Hora de inclusão" type="T" visible="true" useFooter="false"></field>
              <field name="CODUSUINC" label="Usuário de inclusão" type="S" visible="true" useFooter="false"></field>
              <field name="DTALT" label="Data/Hora da últ. alteração" type="T" visible="true" useFooter="false"></field>
              <field name="CODUSUALT" label="Usuário da últ. alteração" type="S" visible="true" useFooter="false"></field>
              <field name="VLRLANC" label="Valor do Lançamento" type="F" visible="true" useFooter="SUM"></field>
            </metadata>
          </grid>
        </container>
      </container>
    </container>
  </level>
</gadget>
```