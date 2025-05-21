```sql
	BEGIN
	  
	VARIAVEIS_PKG.V_ATUALIZANDO := TRUE;
	
	END;
```

```java

BigDecimal qtdApontadaSinal1 = BigDecimalUtil.getValueOrZero(NativeSql.getBigDecimal("SUM(QTD)", "TPRMER", "IDIPROC = ? AND IDIATV = ? AND SINAL = 1", new Object[]{idIproc, idIatv}));

```

````SQL
TESTE
```
----


