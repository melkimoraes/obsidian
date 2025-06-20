Notazz
- [x] excluir CODTIPOPER NFS e COD SERVICO da config geral!
- [x] perguntar pra Marisa qual dos dois campos de cidade do print dela? os dois? pq vi que na integração nao preenchi nenhum deles.

código do serviço 2966
 1 nota por serviço → nao entra pj, exportação, produto

- [ ] criar campo de VLR. → AD_INTGURUAPRVNFSE
- [ ] criar campos na CONFIG GURU → TOP e Serviço da nota unica.
- [x] perguntar pro rogerio do tipo de negociação/parceiro → nota unica
- [ ] fazer algum log da nota unica!! → AD_LOGGURUNU
- [ ] MUDAR PK DA AD_INTGURUAPRVNFSE PRO NROFATURA.
- [ ] sera que consigo testar/acompanhar um cancelamento?
	- [ ] primeiro um cancelamento de nota que vai impactar no valor da nota unica
	- [ ] e outro um cancelamento depois da nota unica emitida! nao é pra cancelar a nota unica!

---

arrumando a prod!

finder com oq tinha colocado no ProcessarNotasGuruNFS:

BigDecimal qtdParaFaturar = qtdFaturar.subtract(qtdFaturados);  
FinderWrapper finderCab = new FinderWrapper("CabecalhoNota",  
        "this.NUNOTA IN (\n"  
                + "  SELECT NUNOTA FROM (\n"  
                + "    SELECT NOTAS.NUNOTA, NOTAS.DTFATUR\n"  
                + "    FROM (\n"  
                + "      SELECT NUNOTA, DTFATUR\n"  
                + "      FROM AD_INTGURUAPRVNFSE\n"  
                + "      WHERE DTFATURADO IS NULL\n"  
                + "        AND NVL(CANCELADO, 'N') = 'N'\n"  
                + "        AND NVL(QTDTNTFAT, 0) < 3\n"  
                + "        AND NUNOTA <> 0\n"  
                + "        AND NVL(VLRFATURA,0) = 0\n"  
                + "    ) NOTAS\n"  
                + "    WHERE DTFATUR <= SYSDATE\n"  
                + "      AND NOTAS.NUNOTA IN (SELECT NUNOTA FROM TGFCAB)\n"  
                + "    ORDER BY NOTAS.DTFATUR ASC\n"  
                + "  ) WHERE ROWNUM <= ?\n"  
                + ")",  
        new Object[]{qtdParaFaturar.compareTo(BigDecimal.ZERO) == -1 ? BigDecimal.ZERO : qtdParaFaturar});  
finderCab.setMaxResults(-1);  
Collection<PersistentLocalEntity> colCab = dwfFacade.findByDynamicFinder(finderCab);