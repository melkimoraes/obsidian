($$F{DESCRICAOCONST} == null ? "" : $F{DESCRICAOCONST})
($F{IDCONFIRMEDDEFECT} == null ? "" : " - ")
$F{IDCONFIRMEDDEFECT}
$F{DESCRICAOCONST}


($F{OPERACAO} == null ? "" : $F{OPERACAO})
---
`JapeSession.SessionHandle hnd = null;
        try {
            hnd = JapeSession.open();
            hnd.execWithTX((JapeSession.TXBlock) new JapeSession.TXBlock() {
                @Override
                public void doWithTx() throws Exception {
				
				}
            });
        } catch (Exception e) {
            System.out.println("br.com.sankhya.gm.action.ImportarNotaExcel.insereLogImportacao()" + e.getMessage());
            throw e;
        } finally {
            JapeSession.close(hnd);
        }`
---

((IF (PESO < 10, 15.10,
 IF (PESO >= 10 AND PESO < 20, 16.04,
   IF (PESO >= 20 AND PESO < 30, 16.61,
    IF (PESO >= 30 AND PESO < 50, 18.02,
     IF (PESO >= 50 AND PESO < 70, 19.89,
      IF (PESO >= 70 AND PESO < 100, 21.83,
       IF (PESO >= 100 AND PESO < 150, 29.93, 38.03)
)))))) /100)* VLRNOTA) + VLRNOTA

