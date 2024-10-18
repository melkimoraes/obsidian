```
<gadget>
  <level id="lvl_wvq2lt" description="Principal">
    <args>
      <arg id="A_LOG" type="text"/>
    </args>
    <container orientacao="H" tamanhoRelativo="100">
      <container orientacao="V" tamanhoRelativo="50">
        <grid id="grd_wvq2lu" useNewGrid="S">
          <expression type="sql" data-source="MGEDS"><![CDATA[SELECT DISTINCT LOG FROM (
SELECT DISTINCT LOG FROM AD_INTNTZNFEA WHERE LOG IS NOT NULL
UNION ALL
SELECT DISTINCT LOG FROM AD_INTNTZNFSA WHERE LOG IS NOT NULL
UNION ALL
SELECT DISTINCT LOG FROM AD_INTNTZNFEC WHERE LOG IS NOT NULL
UNION ALL
SELECT DISTINCT LOG FROM AD_INTNTZNFSC WHERE LOG IS NOT NULL )]]></expression>
          <metadata>
            <field name="LOG" label="LOG" type="S" visible="true" useFooter="false"/>
          </metadata>
          <refresh-details ui-list="grd_wvq2lx">
            <param id="A_LOG">$LOG</param>
          </refresh-details>
        </grid>
      </container>
      <container orientacao="V" tamanhoRelativo="50">
        <grid id="grd_wvq2lx" useNewGrid="S">
          <args>
            <arg id="A_LOG" type="text"/>
          </args>
          <expression type="sql" data-source="MGEDS"><![CDATA[SELECT ID,PAGE,SEQ,LOG FROM AD_INTNTZNFEA WHERE LOG IS NOT NULL AND LOG = :A_LOG
UNION ALL
SELECT ID,PAGE,SEQ,LOG FROM AD_INTNTZNFSA WHERE LOG IS NOT NULL AND LOG = :A_LOG
UNION ALL
SELECT ID,PAGE,SEQ,LOG FROM AD_INTNTZNFEC WHERE LOG IS NOT NULL AND LOG = :A_LOG
UNION ALL
SELECT ID,PAGE,SEQ,LOG FROM AD_INTNTZNFSC WHERE LOG IS NOT NULL AND LOG = :A_LOG]]></expression>
          <metadata>
            <field name="ID" label="ID" type="I" visible="true" useFooter="false"/>
            <field name="PAGE" label="Pagina" type="I" visible="true" useFooter="false"/>
            <field name="SEQ" label="Sequencia" type="I" visible="true" useFooter="false"/>
            <field name="LOG" label="Log" type="S" visible="true" useFooter="false"/>
          </metadata>
        </grid>
      </container>
    </container>
  </level>
</gadget>
```