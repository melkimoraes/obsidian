```
SELECT COUNT(1) FROM TGFCAB WHERE AD_IDPED IS NOT NULL AND NUNOTA>6577128

--6577128

10:34

package br.com.sankhya.gm.integracaoguru.schedule;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.math.BigDecimal;
import java.net.HttpURLConnection;
import java.net.URL;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

import org.cuckoo.core.ScheduledAction;
import org.cuckoo.core.ScheduledActionContext;

import com.google.gson.JsonElement;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;
import com.sankhya.util.FinalWrapper;
import com.sankhya.util.TimeUtils;

import br.com.sankhya.extensions.actionbutton.AcaoRotinaJava;
import br.com.sankhya.extensions.actionbutton.ContextoAcao;
import br.com.sankhya.gm.integracaoguru.model.Pedido;
import br.com.sankhya.gm.integracaoguru.parse.ParsePedido;
import br.com.sankhya.jape.EntityFacade;
import br.com.sankhya.jape.bmp.PersistentLocalEntity;
import br.com.sankhya.jape.core.JapeSession;
import br.com.sankhya.jape.dao.JdbcWrapper;
import br.com.sankhya.jape.sql.NativeSql;
import br.com.sankhya.jape.util.FinderWrapper;
import br.com.sankhya.jape.vo.DynamicVO;
import br.com.sankhya.jape.wrapper.JapeFactory;
import br.com.sankhya.jape.wrapper.JapeWrapper;
import br.com.sankhya.modelcore.auth.AuthenticationInfo;
import br.com.sankhya.modelcore.comercial.BarramentoRegra;
import br.com.sankhya.modelcore.comercial.centrais.CACHelper;
import br.com.sankhya.modelcore.util.EntityFacadeFactory;
import br.com.sankhya.modelcore.util.SPBeanUtils;
import br.com.sankhya.ws.ServiceContext;

public class ProcessarNotasGuru implements ScheduledAction, AcaoRotinaJava {

    private static String TOKEN;
    private JapeSession.SessionHandle hnd = null;

    @Override
    public void onTime(ScheduledActionContext sc) {
        JapeSession.SessionHandle hnd = null;
        ServiceContext sctx = null;
        sctx = new ServiceContext(null);
        sctx.setAutentication(AuthenticationInfo.getCurrent());
        sctx.makeCurrent();

        try {
            SPBeanUtils.setupContext(sctx);
        } catch (Exception e) {
            e.printStackTrace();
            sc.info("Error: Não foi Possivel Executar a Chamada SPBeanUtils.setupContext \n" + e.getMessage());
        }

        try {
            System.out.println("Inicio do Processar Notas - Integração Guru");

            EntityFacade dwfFacade = EntityFacadeFactory.getDWFFacade();

            getToken();

            processarNotas(dwfFacade);

            System.out.println("Fim do Processar Notas - Integração Guru");
        } catch (Exception e) {
            System.out.println("br.com.sankhya.gm.integracaoguru.schedule.ProcessarNotasGuru" + e.getMessage());
            e.printStackTrace();
        }
    }

    @Override
    public void doAction(ContextoAcao ctx) throws Exception {
        System.out.println("Inicio do Processar Notas - Integração Guru");

        TOKEN = ctx.getParametroSistema("TOKENGURU").toString();
        EntityFacade dwfFacade = EntityFacadeFactory.getDWFFacade();

        processarNotas(dwfFacade);

        System.out.println("Fim do Processar Notas - Integração Guru");
    }

    private void getToken() throws Exception {
        try {
            EntityFacade dwf = EntityFacadeFactory.getDWFFacade();
            String sql = "SELECT TEXTO FROM TSIPAR WHERE CHAVE='TOKENGURU'";

            NativeSql sqlBusca = new NativeSql(dwf.getJdbcWrapper());
            sqlBusca.setReuseStatements(false);
            sqlBusca.appendSql(sql);

            ResultSet result = sqlBusca.executeQuery(sql);

            if (result.next()) {
                TOKEN = result.getString("TEXTO");
            }

        } catch (Exception e) {
            System.out.println(
                    "br.com.sankhya.gm.integracaoguru.schedule.ProcessarNotasGuru.getToken()" + e.getMessage());
            throw e;
        }
    }

    private void processarNotas(EntityFacade dwfFacade) throws Exception {
        try {
            JapeWrapper configDAO = JapeFactory.dao("AD_CONFIGINTGURU");
            DynamicVO configVO = configDAO.findOne("this.ID = 1", new Object[]{});

            BigDecimal qtdFaturar = configVO.asBigDecimal("QTDNTFATURFIXO") != null ? configVO.asBigDecimal("QTDNTFATURFIXO") : configVO.asBigDecimal("QTDNTFATUR");

            NativeSql nativeSql = new NativeSql(dwfFacade.getJdbcWrapper());
            String query = "SELECT SUM(QTD) AS QTDTOT FROM (\r\n" + //
                    "    SELECT COUNT(1) AS QTD FROM AD_INTGURUAPRVNFSE WHERE TRUNC(DTFATURADO) = TRUNC(SYSDATE)\r\n" + //
                    "    UNION ALL\r\n" + //
                    "   SELECT COUNT(1) AS QTD FROM AD_INTGURUAPRVNFE WHERE TRUNC(DTFATURADO) = TRUNC(SYSDATE)\r\n" + //
                    ")";
            ResultSet rs = nativeSql.executeQuery(query);
            rs.next();
            BigDecimal qtdFaturados = rs.getBigDecimal("QTDTOT");

            BigDecimal qtdParaFaturar = qtdFaturar.subtract(qtdFaturados);
            FinderWrapper finderCab = new FinderWrapper("CabecalhoNota",
                    "this.NUNOTA IN (SELECT NOTAS.NUNOTA\n"
                    + "  FROM (SELECT NUNOTA, DTFATUR\n"
                    + "          FROM AD_INTGURUAPRVNFSE\n"
                    + "         WHERE DTFATURADO IS NULL\n"
                    + "           AND NVL(CANCELADO, 'N') = 'N'\n"
                    + "           AND NVL(QTDTNTFAT, 0) < 3\n"
                    + "        UNION ALL\n"
                    + "        SELECT NUNOTA, DTFATUR\n"
                    + "          FROM AD_INTGURUAPRVNFE\n"
                    + "         WHERE DTFATURADO IS NULL\n"
                    + "           AND NVL(CANCELADO, 'N') = 'N'\n"
                    + "           AND NVL(QTDTNTFAT, 0) < 3\n"
                    + "         ORDER BY 2) NOTAS\n"
                    + " WHERE ROWNUM <= ?\n"
                    + "   AND DTFATUR <= SYSDATE AND TO_CHAR(DTFATUR, 'MMYYYY') = TO_CHAR(SYSDATE, 'MMYYYY')\n"
                    + "   AND NOTAS.NUNOTA IN (SELECT NUNOTA FROM TGFCAB)\n"
                    + ")",
                    new Object[]{qtdParaFaturar.compareTo(BigDecimal.ZERO) == -1 ? BigDecimal.ZERO : qtdParaFaturar});
            finderCab.setMaxResults(-1);
            Collection<PersistentLocalEntity> colCab = dwfFacade.findByDynamicFinder(finderCab);

            List<String> listPedidoCancelado = new ArrayList<String>();

            ServiceContext sctx = new ServiceContext(null);
            AuthenticationInfo auth = new AuthenticationInfo("SUP", BigDecimal.ZERO, BigDecimal.ZERO, 0);

            sctx.setAutentication(auth);
            sctx.makeCurrent();

            SPBeanUtils.setupContext(sctx);

            CACHelper cacHelper = new CACHelper();
            BarramentoRegra bRegras = BarramentoRegra.build(CACHelper.class, "regrasConfirmacaoCAC.xml",
                    (AuthenticationInfo) sctx.getAutentication());
            int teste = 0;
            for (PersistentLocalEntity cabPLE : colCab) {
                DynamicVO cabVO = (DynamicVO) cabPLE.getValueObject();

                BigDecimal nuNota = cabVO.asBigDecimal("NUNOTA");

                DynamicVO nfVO = JapeFactory.dao("AD_INTGURUAPRVNFE").findOne("this.NUNOTA = ?", nuNota);
                boolean nfse = false;

                if (nfVO == null) {
                    nfVO = JapeFactory.dao("AD_INTGURUAPRVNFSE").findOne("this.NUNOTA = ?", nuNota);
                    nfse = true;
                }

                String idPed = cabVO.asString("AD_IDPED");

                if (!listPedidoCancelado.contains(idPed)) {
//                                if (nfVO.asBigDecimal("NROFATURA").compareTo(BigDecimal.ONE) == 0) {
                    if (cancelada(idPed)) {
                        atualizaNotasCanceladas(idPed);
                        listPedidoCancelado.add(idPed);
                    } else {
//                                }

                        try {
                            dtinicioFaturamento(nfse, nfVO);

                            atualizaDtCab(cabVO);

                            if (!cabVO.asString("STATUSNOTA").equals("L")) {
                                confirmarNf(nuNota, cacHelper, bRegras);
                            } else if (!"A".equals(nfse ? cabVO.asString("STATUSNFSE") : cabVO.asString("STATUSNFE"))) {
                                processarNf(nuNota, cacHelper, bRegras, null);
                            }

                            NativeSql nativesql2 = new NativeSql(dwfFacade.getJdbcWrapper());
                            nativesql2.setNamedParameter("NUNOTA",nuNota);
                            ResultSet rs2 = nativesql2.executeQuery("SELECT STATUSNOTA, STATUSNFE, STATUSNFSE, (SELECT OCORRENCIAS FROM TGFACT WHERE NUNOTA=TGFCAB.NUNOTA AND SEQUENCIA = (SELECT MAX(SEQUENCIA) FROM TGFACT WHERE NUNOTA = TGFCAB.NUNOTA)) AS OCORRENCIA FROM TGFCAB WHERE NUNOTA = :NUNOTA");

                            if (rs2.next()) {
                                if ("A".equals(nfse ? rs2.getString("STATUSNFSE") : rs2.getString("STATUSNFE"))) {
                                    atualizaNfGuru(nfse, nfVO);
                                } else {
                                    throw new Exception(rs2.getString("OCORRENCIA"));
                                }
                            }

                        } catch (Exception e) {
                            //throw e;
                            insereLogFatur(nfse, nfVO, e.toString());
                        }
                    }
                }
            }
        } catch (Exception e) {
            System.out.println(
                    "br.com.sankhya.gm.integracaoguru.schedule.ProcessarNotasGuru.processarNotas()" + e.getMessage());
            e.printStackTrace();
        }
    }

    private void dtinicioFaturamento(boolean nfse, DynamicVO nfVO) throws Exception {
        JapeSession.SessionHandle hnd = null;
        try {
            hnd = JapeSession.open();
            hnd.execWithTX((JapeSession.TXBlock) new JapeSession.TXBlock() {
                @Override
                public void doWithTx() throws Exception {
                    if (nfse) {
                        JapeWrapper intGuruDAO = JapeFactory.dao("AD_INTGURUAPRVNFSE");
                        intGuruDAO.prepareToUpdate(nfVO)
                                .set("DTINIFATUR", TimeUtils.getNow())
                                .update();
                    } else {
                        JapeWrapper intGuruDAO = JapeFactory.dao("AD_INTGURUAPRVNFE");
                        intGuruDAO.prepareToUpdate(nfVO)
                                .set("DTINIFATUR", TimeUtils.getNow())
                                .update();
                    }
                }
            });
        } catch (Exception e) {
            System.out.println("br.com.sankhya.gm.integracaoguru.ProcessarNotasGuru.dtinicioFaturamento()" + e.getMessage());
            throw e;
        } finally {
            JapeSession.close(hnd);
        }
    }

    private void atualizaDtCab(DynamicVO cabVO) throws Exception {
        JapeSession.SessionHandle hnd = null;
        try {
            hnd = JapeSession.open();
            hnd.execWithTX((JapeSession.TXBlock) new JapeSession.TXBlock() {
                @Override
                public void doWithTx() throws Exception {
                    JapeWrapper cabDAO = JapeFactory.dao("CabecalhoNota");
                    cabDAO.prepareToUpdate(cabVO)
                            .set("DTNEG", TimeUtils.getNow())
                            .set("DTFATUR", TimeUtils.getNow())
                            .update();
                }
            });
        } catch (Exception e) {
            System.out.println("br.com.sankhya.gm.integracaoguru.ProcessarNotasGuru.dtinicioFaturamento()" + e.getMessage());
            throw e;
        } finally {
            JapeSession.close(hnd);
        }
    }

    private void atualizaNfGuru(boolean nfse, DynamicVO nfVO) throws Exception {
        JapeSession.SessionHandle hnd = null;
        try {
            hnd = JapeSession.open();
            hnd.execWithTX((JapeSession.TXBlock) new JapeSession.TXBlock() {
                @Override
                public void doWithTx() throws Exception {
                    if (nfse) {
                        JapeWrapper intGuruDAO = JapeFactory.dao("AD_INTGURUAPRVNFSE");
                        intGuruDAO.prepareToUpdate(nfVO)
                                .set("DTFATURADO", TimeUtils.getNow())
                                .set("FATURADO", "S")
                                .update();
                    } else {
                        JapeWrapper intGuruDAO = JapeFactory.dao("AD_INTGURUAPRVNFE");
                        intGuruDAO.prepareToUpdate(nfVO)
                                .set("DTFATURADO", TimeUtils.getNow())
                                .set("FATURADO", "S")
                                .update();
                    }
                }
            });
        } catch (Exception e) {
            System.out.println("br.com.sankhya.gm.integracaoguru.ProcessarNotasGuru.atualizaNfGuru()" + e.getMessage());
            throw e;
        } finally {
            JapeSession.close(hnd);
        }
    }

    private void insereLogFatur(boolean nfse, DynamicVO nfVO, String message) throws Exception {
        JapeSession.SessionHandle hnd = null;
        try {
            hnd = JapeSession.open();
            hnd.execWithTX((JapeSession.TXBlock) new JapeSession.TXBlock() {
                @Override
                public void doWithTx() throws Exception {
                    BigDecimal qtdTnt = nfVO.asBigDecimal("QTDTNTFAT") == null ? BigDecimal.ZERO
                            : nfVO.asBigDecimal("QTDTNTFAT");
                    if (nfse) {
                        JapeWrapper intGuruDAO = JapeFactory.dao("AD_INTGURUAPRVNFSE");
                        intGuruDAO.prepareToUpdate(nfVO)
                                .set("LOGFAT", message)
                                .set("QTDTNTFAT", qtdTnt.add(BigDecimal.ONE))
                                .update();
                    } else {
                        JapeWrapper intGuruDAO = JapeFactory.dao("AD_INTGURUAPRVNFE");
                        intGuruDAO.prepareToUpdate(nfVO)
                                .set("LOGFAT", message)
                                .set("QTDTNTFAT", qtdTnt.add(BigDecimal.ONE))
                                .update();
                    }
                }
            });
        } catch (Exception e) {
            System.out.println("br.com.sankhya.gm.integracaoguru.ProcessarNotasGuru.insereLogFatur()" + e.getMessage());
            throw e;
        } finally {
            JapeSession.close(hnd);
        }
    }

    private void confirmarNf(BigDecimal nuNota, CACHelper cacHelper, BarramentoRegra bRegras) throws Exception {
        FinalWrapper<?> cabState = cacHelper.confirmarNota(nuNota, bRegras);
        processarNf(nuNota, cacHelper, bRegras, cabState);
//        cacHelper.processarNotaConfirmada(nuNota, bRegras, cabState);
    }

    private void processarNf(BigDecimal nuNota, CACHelper cacHelper, BarramentoRegra bRegras, FinalWrapper<?> cabState) throws Exception {
        cacHelper.processarNotaConfirmada(nuNota, bRegras, cabState);
    }

    private void atualizaNotasCanceladas(String idPed) throws Exception {
        JapeSession.SessionHandle hnd = null;
        try {
            hnd = JapeSession.open();
            hnd.execWithTX((JapeSession.TXBlock) new JapeSession.TXBlock() {
                @Override
                public void doWithTx() throws Exception {
                    EntityFacade dwf = EntityFacadeFactory.getDWFFacade();
                    JdbcWrapper jdbc = dwf.getJdbcWrapper();
                    jdbc.openSession();
                    NativeSql nativeSql = new NativeSql(jdbc);
                    nativeSql.setNamedParameter("IDPED", idPed);
                    nativeSql.executeUpdate("UPDATE AD_INTGURUAPRVNFE  SET CANCELADO = 'S', DTCANCEL=SYSDATE WHERE IDPED = :IDPED AND NVL(FATURADO,'N') != 'S'");
                    nativeSql.executeUpdate("UPDATE AD_INTGURUAPRVNFSE  SET CANCELADO = 'S', DTCANCEL=SYSDATE WHERE IDPED = :IDPED AND NVL(FATURADO,'N') != 'S'");

                    String query = "SELECT NUNOTA FROM AD_INTGURUAPRVNFE WHERE CANCELADO = 'S' AND IDPED = :IDPED\n"
                            + "UNION ALL\n"
                            + "SELECT NUNOTA FROM AD_INTGURUAPRVNFSE WHERE CANCELADO = 'S' AND IDPED = :IDPED";
                    ResultSet rs = nativeSql.executeQuery(query);
                    while (rs.next()) {
                        BigDecimal nuNota = rs.getBigDecimal("NUNOTA");
                        JapeWrapper cabDAO = JapeFactory.dao("CabecalhoNota");
                        cabDAO.deleteByCriteria("this.NUNOTA = ?", new Object[]{nuNota});
                    }
                }
            });
        } catch (Exception e) {
            System.out.println(
                    "br.com.sankhya.gm.integracaoguru.ProcessarNotasGuru.atualizaNotasCanceladas()" + e.getMessage());
            throw e;
        } finally {
            JapeSession.close(hnd);
        }
    }

    private boolean cancelada(String idPed) throws Exception {
        String response = getJsonResponse(
                "https://digitalmanager.guru/api/v1/transactions/" + idPed);

        Pedido responsePedido = ParsePedido.fromJson(response);
        return responsePedido.dates.canceled_at != 0;
    }

    private String getJsonResponse(String url) throws Exception {
        try {
            HttpURLConnection connection = (HttpURLConnection) new URL(url).openConnection();
            connection.setRequestMethod("GET");
            connection.setRequestProperty("Content-Type", "application/json");
            connection.setRequestProperty("Accept", "application/json");
            connection.setRequestProperty("Authorization", "Bearer " + TOKEN);

            try (BufferedReader br = new BufferedReader(
                    new InputStreamReader(connection.getInputStream()))) {
                StringBuilder response = new StringBuilder();
                String output;
                while ((output = br.readLine()) != null) {
                    response.append(output);
                }
                return response.toString();
            }
        } catch (Exception e) {
            throw e;
        }
    }

}

```