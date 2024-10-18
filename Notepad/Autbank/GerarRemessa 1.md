```
package br.com.sankhya.gm.autbank.action;

import static br.com.sankhya.gm.autbank.util.StringUtil.toTimestamp;

import java.math.BigDecimal;
import java.math.RoundingMode;
import java.sql.ResultSet;
import java.sql.Timestamp;
import java.util.ArrayList;
import java.util.List;

import com.sankhya.util.TimeUtils;

import br.com.sankhya.extensions.actionbutton.AcaoRotinaJava;
import br.com.sankhya.extensions.actionbutton.ContextoAcao;
import br.com.sankhya.extensions.actionbutton.QueryExecutor;
import br.com.sankhya.gm.autbank.model.Financeiro;
import br.com.sankhya.jape.EntityFacade;
import br.com.sankhya.jape.core.JapeSession;
import br.com.sankhya.jape.core.JapeSession.SessionHandle;
import br.com.sankhya.jape.sql.NativeSql;
import br.com.sankhya.jape.vo.DynamicVO;
import br.com.sankhya.jape.wrapper.JapeFactory;
import br.com.sankhya.jape.wrapper.JapeWrapper;
import br.com.sankhya.jape.wrapper.fluid.FluidCreateVO;
import br.com.sankhya.modelcore.util.EntityFacadeFactory;

public class GerarRemessa implements AcaoRotinaJava {

        String dtIni = "";
        String dtFin = "";
        BigDecimal codEmp;
        List<Financeiro> listFin;
        BigDecimal numRem = null;
        BigDecimal id = null;

        @Override
        public void doAction(ContextoAcao ctx) throws Exception {
                SessionHandle hnd = null;

                try {
                        hnd = JapeSession.open();

                        dtIni = TimeUtils.formataDDMMYY((Timestamp) ctx.getParam("DTINI"));
                        dtFin = TimeUtils.formataDDMMYY((Timestamp) ctx.getParam("DTFIN"));
                        codEmp = new BigDecimal((String) ctx.getParam("CODEMP"));

                        if (!dtIni.substring(3, 5).equals(dtFin.substring(3, 5))
                                        || !dtIni.substring(6, 8).equals(dtFin.substring(6, 8)))
                                throw new Exception("Informar periodo do mesmo mês!");

                        DynamicVO remessaVO = JapeFactory.dao("AD_INTAUTBANK").findOne(
                                        "onlyDate(this.DTINI) >= ? AND onlyDate(this.DTFIN) <= ? AND CODEMP = ?",
                                        new Object[] { dtIni, dtFin, codEmp });

                        if (remessaVO != null)
                                throw new Exception("Já existe Remessa gerada nesse periodo para a Empresa " + codEmp
                                                + ". Nro "
                                                + remessaVO.getProperty("NUMREM"));

                        listFin = new ArrayList<Financeiro>();

                        // Lançamentos as Fornecedores (COMUM)
                        // Lançamento de Saida
                        buscaFinanceiroLiquidacao(ctx);

                        // Lançamento de Provisao
                        buscaFinanceiroProvisao(ctx);

                        // Lançamentos de Contratos
                        buscaFinanceiroLiquidacaoContratos(ctx);
                        buscaFinanceiroProvisaoContratos(ctx);
                        buscaDiferimentoContratos(ctx);

                        // movimentos bancarios
                        buscaMovimentosBancarios(ctx);

                        inserirRemessa();

                        ctx.setMensagemRetorno("Remessa gerada com sucesso. Nro " + numRem + " Empresa " + codEmp);

                } catch (Exception e) {
                        e.printStackTrace();
                        throw new Exception(e);
                } finally {
                        JapeSession.close(hnd);
                }
        }

        private void buscaMovimentosBancarios(ContextoAcao ctx) throws Exception {
                QueryExecutor query = ctx.getQuery();
                query.setParam("DTINI", dtIni);
                query.setParam("DTFIN", dtFin);
                query.setParam("CODEMP", codEmp);

                query.nativeSelect("SELECT \r\n" + //
                                "    MBC.NUBCO,\r\n" + //
                                "    MBC.VLRLANC,\r\n" + //
                                "    CTA.AD_CODCTACT,\r\n" + //
                                "    MBC.CODCTABCOINT,\r\n" + //
                                "    MBC.HISTORICO,\r\n" + //
                                "    CONVERT(VARCHAR(8), MBC.DHCONCILIACAO, 3) AS DHCONCILIACAO\r\n" + //
                                "FROM TGFMBC MBC\r\n" + //
                                "INNER JOIN TSICTA CTA ON CTA.CODCTABCOINT = MBC.CODCTABCOINT\r\n" + //
                                "WHERE onlyDate(MBC.DHCONCILIACAO) >= {DTINI}\r\n" + //
                                "AND onlyDate(MBC.DHCONCILIACAO) <= {DTFIN}\r\n" + //
                                "AND CTA.CODEMP = {CODEMP}\r\n" + //
                                "AND MBC.RECDESP = -1\r\n" + //
                                "AND ORIGMOV IN ('T','R','A')");

                while (query.next()) {
                        String complemento = query.getString("HISTORICO");

                        DynamicVO mbcVO = JapeFactory.dao("MovimentoBancario").findByPK(query.getBigDecimal("NUBCO"));
                        DynamicVO ctaVO = JapeFactory.dao("ContaBancaria")
                                        .findByPK(mbcVO.asBigDecimal("CODCTABCOINTDEST"));

                        Financeiro fin = new Financeiro();
                        fin.setNuFin(query.getBigDecimal("NUBCO"));
                        fin.setVlrDesdob(query.getBigDecimal("VLRLANC"));
                        fin.setComplemento(complemento);
                        fin.setCtaCtb(query.getString("AD_CODCTACT"));
                        fin.setCtaCtbPtd(ctaVO.asString("AD_CODCTACT"));
                        fin.setDt(TimeUtils.toTimestamp(query.getString("DHCONCILIACAO"), "dd/MM/yy"));
                        fin.setLancamento("M");
                        listFin.add(fin);

                }
                query.close();
        }

        private void inserirRemessa() throws Exception {
                FluidCreateVO remessaFVO = JapeFactory.dao("AD_INTAUTBANK").create();
                DynamicVO empVO = JapeFactory.dao("Empresa").findByPK(codEmp);

                numRem = getNumRem();

                DynamicVO remessaVO = remessaFVO
                                .set("DTINI", toTimestamp(dtIni, "dd/MM/yy"))
                                .set("DTFIN", toTimestamp(dtFin, "dd/MM/yy"))
                                .set("CODEMP", codEmp)
                                .set("NUMREM", numRem)
                                .set("EMPRESA", empVO.asBigDecimal("AD_CODEMPAUTBANK"))
                                .set("UNIDADE", empVO.asBigDecimal("AD_CODUNIDAUTBANK"))
                                .set("QTDLANC", new BigDecimal(listFin.size() * 2))
                                .save();

                id = (BigDecimal) remessaVO.getProperty("ID");

                FluidCreateVO lancamentoFVO = JapeFactory.dao("AD_LANCINTAUTBANK").create();
                FluidCreateVO contrLancFVO = JapeFactory.dao("AD_CONLANCAUTBANK").create();
                JapeWrapper lancDAO = JapeFactory.dao("AD_LANCINTAUTBANK");
                JapeWrapper contrLancDAO = JapeFactory.dao("AD_CONLANCAUTBANK");
                int lanc = 1;
                for (int i = 0; i < listFin.size(); i++) {
                        Financeiro fin = listFin.get(i);
                        int item = 1;
                        if (!fin.getContrato()) {
                                if ("S".equals(fin.getLancamento()) || "E".equals(fin.getLancamento())) {
                                        lancDAO.create()
                                                        .set("ID", id)
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("TIPLANC", "S")
                                                        .set("CONTA", fin.getRecDesp().compareTo(BigDecimal.ONE) == 0
                                                                        ? fin.getCtaCtbPtd()
                                                                        : fin.getCtaCtb()) // //fin.getVlrDesdob() ->
                                                                                           // recdesp = 1 RECEITA
                                                        .set("VLRDEB", fin.getVlrDesdob())
                                                        .set("VLRCRED", null)
                                                        .set("CODCENCUS", fin.getCodCencus())
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("LANC", fin.getLancamento())
                                                        .set("DTLANC", fin.getDt())
                                                        .save();
                                        lanc++;
                                        // item++;
                                        lancDAO.create()
                                                        .set("ID", id)
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("TIPLANC", "S")
                                                        .set("CONTA", fin.getRecDesp().compareTo(BigDecimal.ONE) == 0
                                                                        ? fin.getCtaCtb()
                                                                        : fin.getCtaCtbPtd()) // // fin.getCtaCtb()
                                                        .set("VLRDEB", null)
                                                        .set("VLRCRED", fin.getVlrDesdob())
                                                        .set("CODCENCUS", fin.getCodCencus())
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("LANC", fin.getLancamento())
                                                        .set("DTLANC", fin.getDt())
                                                        .save();
                                        lanc++;

                                } else if ("PE".equals(fin.getLancamento()) || "PS".equals(fin.getLancamento())) {
                                        lancDAO.create()
                                                        .set("ID", id)
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("TIPLANC", "S")
                                                        .set("CONTA", fin.getRecDesp().compareTo(BigDecimal.ONE) == 0
                                                                        ? fin.getCtaCtbPtd()
                                                                        : fin.getCtaCtb()) // fin.getCtaCtb() //
                                                        // fin.getRecDesp().compareTo(BigDecimal.ONE)
                                                        // == 0 ? fin.getCtaCtb() :
                                                        // fin.getCtaCtbPtd()
                                                        .set("VLRDEB", fin.getVlrDesdob())
                                                        .set("VLRCRED", null)
                                                        .set("CODCENCUS", fin.getCodCencus())
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("LANC", fin.getLancamento())
                                                        .set("DTLANC", fin.getDt())
                                                        .save();
                                        lanc++;
                                        // item++;
                                        lancDAO.create()
                                                        .set("ID", id)
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("TIPLANC", "S")
                                                        .set("CONTA", fin.getRecDesp().compareTo(BigDecimal.ONE) == 0
                                                                        ? fin.getCtaCtb()
                                                                        : fin.getCtaCtbPtd())// fin.getCtaCtbPtd() //
                                                                                             // fin.getRecDesp().compareTo(BigDecimal.ONE)
                                                                                             // == 0 ? getCtaCtbPtd() :
                                                                                             // fin.getCtaCtb()
                                                        .set("VLRDEB", null)
                                                        .set("VLRCRED", fin.getVlrDesdob())
                                                        .set("CODCENCUS", fin.getCodCencus())
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("LANC", fin.getLancamento())
                                                        .set("DTLANC", fin.getDt())
                                                        .save();
                                        lanc++;
                                } else if ("M".equals(fin.getLancamento())) {
                                        lancDAO.create()
                                                        .set("ID", id)
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("TIPLANC", "S")
                                                        .set("CONTA", fin.getCtaCtbPtd()) // //fin.getVlrDesdob() ->
                                                                                          // recdesp = 1 RECEITA
                                                        .set("VLRDEB", fin.getVlrDesdob())
                                                        .set("VLRCRED", null)
                                                        .set("CODCENCUS", fin.getCodCencus())
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("LANC", fin.getLancamento())
                                                        .set("DTLANC", fin.getDt())
                                                        .save();
                                        lanc++;
                                        // item++;
                                        lancDAO.create()
                                                        .set("ID", id)
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("TIPLANC", "S")
                                                        .set("CONTA",
                                                                        fin.getCtaCtb()) // // fin.getCtaCtb()
                                                        .set("VLRDEB", null)
                                                        .set("VLRCRED", fin.getVlrDesdob())
                                                        .set("CODCENCUS", fin.getCodCencus())
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("LANC", fin.getLancamento())
                                                        .set("DTLANC", fin.getDt())
                                                        .save();
                                        lanc++;
                                }
                        } else {
                                if ("S".equals(fin.getLancamento()) || "E".equals(fin.getLancamento())) {
                                        lancDAO.create()
                                                        .set("ID", id)
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("TIPLANC", "S")
                                                        .set("CONTA", fin.getRecDesp().compareTo(BigDecimal.ONE) == 0
                                                                        ? fin.getCtaCtbPtd()
                                                                        : fin.getCtaCtb())
                                                        .set("VLRDEB", fin.getVlrDesdob())
                                                        .set("VLRCRED", null)
                                                        .set("CODCENCUS", fin.getCodCencus())
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("LANC", fin.getLancamento())
                                                        .set("DTLANC", fin.getDt())
                                                        .set("LANCCONTRATO", "S")
                                                        .set("NUMCONTRATO", fin.getNumContrato())
                                                        .save();
                                        lanc++;
                                        // item++;
                                        lancDAO.create()
                                                        .set("ID", id)
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("TIPLANC", "S")
                                                        .set("CONTA", fin.getRecDesp().compareTo(BigDecimal.ONE) == 0
                                                                        ? fin.getCtaCtb()
                                                                        : fin.getCtaCtbPtd())
                                                        .set("VLRDEB", null)
                                                        .set("VLRCRED", fin.getVlrDesdob())
                                                        .set("CODCENCUS", fin.getCodCencus())
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("DTLANC", fin.getDt())
                                                        .set("LANCCONTRATO", "S")
                                                        .set("NUMCONTRATO", fin.getNumContrato())
                                                        .set("LANC", fin.getLancamento())
                                                        .save();
                                        lanc++;

                                } else if ("PE".equals(fin.getLancamento()) || "PS".equals(fin.getLancamento())) {
                                        lancDAO.create()
                                                        .set("ID", id)
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("TIPLANC", "S")
                                                        .set("CONTA", fin.getRecDesp().compareTo(BigDecimal.ONE) == 0
                                                                        ? fin.getCtaCtbPtd()
                                                                        : fin.getCtaCtb()) // fin.getCtaCtbPtd())
                                                        .set("VLRDEB", fin.getVlrDesdob())
                                                        .set("VLRCRED", null)
                                                        .set("CODCENCUS", fin.getCodCencus())
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("LANC", fin.getLancamento())
                                                        .set("DTLANC", fin.getDt())
                                                        .set("LANCCONTRATO", "S")
                                                        .set("NUMCONTRATO", fin.getNumContrato())
                                                        .save();
                                        lanc++;
                                        // item++;
                                        lancDAO.create()
                                                        .set("ID", id)
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("TIPLANC", "S")
                                                        .set("CONTA", fin.getRecDesp().compareTo(BigDecimal.ONE) == 0
                                                                        ? fin.getCtaCtb()
                                                                        : fin.getCtaCtbPtd())
                                                        // fin.getCtaCtb()
                                                        .set("VLRDEB", null)
                                                        .set("VLRCRED", fin.getVlrDesdob())
                                                        .set("CODCENCUS", fin.getCodCencus())
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("DTLANC", fin.getDt())
                                                        .set("LANCCONTRATO", "S")
                                                        .set("NUMCONTRATO", fin.getNumContrato())
                                                        .set("LANC", fin.getLancamento())
                                                        .save();
                                        lanc++;
                                } else if ("D".equals(fin.getLancamento())) {
                                        BigDecimal meses = null;
                                        switch (fin.getPeriodoContrato()) {
                                                case "A":
                                                        meses = new BigDecimal("12");
                                                        break;
                                                case "B":
                                                        meses = new BigDecimal("2");
                                                        break;
                                                case "Q":
                                                        meses = new BigDecimal("4");
                                                        break;
                                                case "S":
                                                        meses = new BigDecimal("6");
                                                        break;
                                                case "T":
                                                        meses = new BigDecimal("3");
                                                        break;
                                        }
                                        BigDecimal vlr = fin.getVlrDesdob().divide(meses, 2, RoundingMode.HALF_UP);

                                        DynamicVO lancVO = lancDAO.create()
                                                        .set("ID", id)
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("TIPLANC", "S")
                                                        .set("CONTA", fin.getCtaCtb())
                                                        .set("VLRDEB", vlr)
                                                        .set("VLRCRED", null)
                                                        .set("CODCENCUS", fin.getCodCencus())
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("LANC", fin.getLancamento())
                                                        .set("DTLANC", fin.getDt())
                                                        .set("LANCCONTRATO", "S")
                                                        .set("NUMCONTRATO", fin.getNumContrato())
                                                        .save();

                                        contrLancDAO.create()
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("SEQ", lancVO.asBigDecimal("SEQ"))
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("CONTA", fin.getCtaCtb())
                                                        .set("VLRDEB", vlr)
                                                        .set("VLRCRED", null)
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("LANC", fin.getLancamento())
                                                        .set("DTLANC", fin.getDt())
                                                        .set("NUMCONTRATO", fin.getNumContrato())
                                                        .save();
                                        lanc++;

                                        // item++;
                                        lancVO = lancDAO.create()
                                                        .set("ID", id)
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("TIPLANC", "S")
                                                        .set("CONTA", fin.getCtaCtbPtd())
                                                        .set("VLRDEB", null)
                                                        .set("VLRCRED", vlr)
                                                        .set("CODCENCUS", fin.getCodCencus())
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("DTLANC", fin.getDt())
                                                        .set("LANCCONTRATO", "S")
                                                        .set("NUMCONTRATO", fin.getNumContrato())
                                                        .set("LANC", fin.getLancamento())
                                                        .save();

                                        contrLancDAO.create()
                                                        .set("NUMREM", numRem)
                                                        .set("CODEMP", codEmp)
                                                        .set("SEQ", lancVO.asBigDecimal("SEQ"))
                                                        .set("NUFIN", fin.getNuFin())
                                                        .set("NROLANC", new BigDecimal(lanc))
                                                        .set("NROITEM", new BigDecimal(item))
                                                        .set("CONTA", fin.getCtaCtbPtd())
                                                        .set("VLRDEB", null)
                                                        .set("VLRCRED", vlr)
                                                        .set("COMPLEMENTOS", fin.getComplemento())
                                                        .set("LANC", fin.getLancamento())
                                                        .set("DTLANC", fin.getDt())
                                                        .set("NUMCONTRATO", fin.getNumContrato())
                                                        .save();
                                        lanc++;
                                }
                        }

                }
        }

        private BigDecimal getNumRem() throws Exception {
                BigDecimal numRem = BigDecimal.ONE;
                EntityFacade dwf = EntityFacadeFactory.getDWFFacade();
                NativeSql nativeSql = new NativeSql(dwf.getJdbcWrapper());

                String query = "SELECT COUNT(1) AS QTD FROM AD_INTAUTBANK WHERE CODEMP = :CODEMP";
                nativeSql.setNamedParameter("CODEMP", codEmp);
                ResultSet rs = nativeSql.executeQuery(query);
                rs.next();
                if (rs.getBigDecimal("QTD").compareTo(BigDecimal.ZERO) != 0) {
                        NativeSql nativeSql2 = new NativeSql(dwf.getJdbcWrapper());
                        nativeSql2.setNamedParameter("CODEMP", codEmp);
                        ResultSet rs2 = nativeSql2.executeQuery(
                                        "SELECT MAX(NUMREM)+1 AS NUMREM FROM AD_INTAUTBANK WHERE CODEMP = :CODEMP");
                        rs2.next();
                        numRem = rs2.getBigDecimal("NUMREM");
                }
                return numRem;
        }

        private void buscaFinanceiroLiquidacao(ContextoAcao ctx) throws Exception {
                QueryExecutor query = ctx.getQuery();

                query.setParam("DTINI", dtIni);
                query.setParam("DTFIN", dtFin);
                query.setParam("CODEMP", codEmp);

                query.nativeSelect("SELECT FIN.NUFIN,\r\n" + //
                                "      FIN.NUMNOTA,\r\n" + //
                                "      RTRIM(PAR.NOMEPARC) AS NOMEPARC,\r\n" + //
                                "      FIN.VLRDESDOB,\r\n" + //
                                "      FIN.VLRJURO,\r\n" + //
                                "      FIN.VLRMULTA,\r\n" + //
                                "      FIN.VLRDESC,\r\n" + //
                                "      FIN.RECDESP,\r\n" + //
                                "      RTRIM(nullValue(CAST(CAB.OBSERVACAO AS VARCHAR(MAX)), FIN.HISTORICO)) AS OBSERVACAO,\r\n"
                                + //
                                "      CUS.AD_CODCENCUSATBK,\r\n" + //
                                "      CTA.AD_CODCTACT,\r\n" + //
                                "      NAT.AD_CONTRAPARTICA,\r\n" + //
                                "      CTA.AD_CODCTACT,\r\n" + //
                                "      NAT.AD_CONTACONTAJM,\r\n" + //
                                "      NAT.AD_CONTACONTADESC,\r\n" + //
                                "      MBC.DHCONCILIACAO,\r\n" + //
                                "    (\r\n" + //
                                "        SELECT STRING_AGG(PRO.DESCRPROD, ', ') \r\n" + //
                                "        FROM TGFITE ITE\r\n" + //
                                "        JOIN TGFPRO PRO ON ITE.CODPROD = PRO.CODPROD\r\n" + //
                                "        WHERE ITE.NUNOTA = CAB.NUNOTA\r\n" + //
                                "    ) AS DESCRPROD,\r\n" + //
                                "    (SELECT COUNT(1) FROM TGFRAT WHERE NUFIN = FIN.NUFIN AND ORIGEM='F') AS QTD\r\n" + //
                                " FROM TGFFIN FIN\r\n" + //
                                " LEFT JOIN TGFCAB CAB\r\n" + //
                                "   ON CAB.NUNOTA = FIN.NUNOTA\r\n" + //
                                "INNER JOIN TGFNAT NAT\r\n" + //
                                "   ON NAT.CODNAT = FIN.CODNAT\r\n" + //
                                "LEFT JOIN TSICUS CUS\r\n" + //
                                "   ON CUS.CODCENCUS = FIN.CODCENCUS\r\n" + //
                                "LEFT JOIN TSICTA CTA\r\n" + //
                                "   ON CTA.CODCTABCOINT = FIN.CODCTABCOINT\r\n" + //
                                "INNER JOIN TGFPAR PAR\r\n" + //
                                "   ON PAR.CODPARC = FIN.CODPARC\r\n" + //
                                "LEFT JOIN TCSCON CON\r\n" + //
                                "   ON CON.NUMCONTRATO = CAB.NUMCONTRATO\r\n" + //
                                "INNER JOIN TGFMBC MBC\r\n" + //
                                "   ON MBC.NUBCO = FIN.NUBCO\r\n" + //
                                "WHERE onlyDate(MBC.DHCONCILIACAO) >= {DTINI}\r\n" + //
                                "  AND onlyDate(MBC.DHCONCILIACAO) <= {DTFIN}\r\n" + //
                                "  AND NAT.AD_CONTABILIZAVEL = 'S'\r\n" + //
                                "  AND ((nullValue(CAB.NUMCONTRATO,0) = 0) OR (CAB.NUMCONTRATO <> 0 AND CON.TIPO = 'L') OR (CAB.NUMCONTRATO <> 0 AND CON.TIPO = 'M'))\r\n"
                                + //
                                "  AND FIN.CODEMP = {CODEMP}");

                while (query.next()) {
                        if(query.getBigDecimal("QTD").compareTo(BigDecimal.ZERO) != 0 && query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) != 0){
                                QueryExecutor query2 = ctx.getQuery();

                                query2.setParam("NUFIN", query.getBigDecimal("NUFIN"));

                                query2.nativeSelect("SELECT \n" + //
                                                "    RAT.NUFIN,\n" + //
                                                "    ROUND(FIN.VLRDESDOB*(RAT.PERCRATEIO/100),2) AS VLR,\n" + //
                                                "    NAT.DESCRNAT AS COMPLEMENTO,\n" + //
                                                "    CUS.AD_CODCENCUSATBK,\n" + //
                                                "    RAT.CODNAT,\n" + //
                                                "    NAT.AD_CONTRAPARTICA,\n" + //
                                                "    CTA.AD_CODCTACT,\n" + //
                                                "    FIN.RECDESP,\n" + //
                                                "    MBC.DHCONCILIACAO\n" + //
                                                "FROM TGFRAT RAT\n" + //
                                                "INNER JOIN TGFFIN FIN ON FIN.NUFIN = RAT.NUFIN\n" + //
                                                "INNER JOIN TGFNAT NAT ON NAT.CODNAT = RAT.CODNAT\n" + //
                                                "LEFT JOIN TSICUS CUS ON CUS.CODCENCUS = FIN.CODCENCUS\n" + //
                                                "LEFT JOIN TSICTA CTA ON CTA.CODCTABCOINT = FIN.CODCTABCOINT\n" + //
                                                "INNER JOIN TGFMBC MBC ON MBC.NUBCO = FIN.NUBCO\n" + //
                                                "WHERE RAT.NUFIN={NUFIN} AND RAT.ORIGEM='F'");
                                while (query2.next()) {
                                        Financeiro fin4 = new Financeiro();
                                        fin4.setNuFin(query2.getBigDecimal("NUFIN"));
                                        fin4.setVlrDesdob(query2.getBigDecimal("VLR"));
                                        fin4.setComplemento(query2.getString("COMPLEMENTO"));
                                        fin4.setCodCencus(query2.getString("AD_CODCENCUSATBK"));
                                        fin4.setCtaCtb(query2.getString("AD_CONTRAPARTICA"));
                                        fin4.setCtaCtbPtd(query2.getString("AD_CODCTACT"));
                                        fin4.setRecDesp(query2.getBigDecimal("RECDESP"));
                                        fin4.setDt(query2.getTimestamp("DHCONCILIACAO"));
                                        fin4.setLancamento(
                                                        query2.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0
                                                                        ? "E"
                                                                        : "S");
                                        listFin.add(fin4);
                                }
                                query2.close();
                        }else{
                                String complemento = "";
                        if (query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0) {
                                complemento = query.getString("OBSERVACAO");
                                complemento = query.getString("DESCRPROD") != null
                                                ? query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC")
                                                                + " - " + query.getString("DESCRPROD") + " - "
                                                                + complemento
                                                : query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC")
                                                                + " - " + complemento;
                                complemento = query.getString("AD_CODCENCUSATBK") != null
                                                ? query.getString("AD_CODCENCUSATBK") + " - " + complemento
                                                : complemento;
                        } else {
                                complemento = query.getString("OBSERVACAO");
                                complemento = complemento != null
                                                ? query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC")
                                                                + " - "
                                                                + complemento
                                                : query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC");
                                complemento = query.getString("AD_CODCENCUSATBK") != null
                                                ? query.getString("AD_CODCENCUSATBK") + " - " + complemento
                                                : complemento;
                        }

                        Financeiro fin = new Financeiro();
                        fin.setNuFin(query.getBigDecimal("NUFIN"));
                        fin.setVlrDesdob(query.getBigDecimal("VLRDESDOB"));
                        fin.setComplemento(complemento);
                        fin.setCodCencus(query.getString("AD_CODCENCUSATBK"));
                        fin.setCtaCtb(query.getString("AD_CONTRAPARTICA"));
                        fin.setCtaCtbPtd(query.getString("AD_CODCTACT"));
                        fin.setRecDesp(query.getBigDecimal("RECDESP"));
                        fin.setDt(query.getTimestamp("DHCONCILIACAO"));
                        fin.setLancamento(query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0 ? "E" : "S");
                        listFin.add(fin);

                        if (query.getBigDecimal("VLRJURO").compareTo(BigDecimal.ZERO) != 0
                                        || query.getBigDecimal("VLRMULTA").compareTo(BigDecimal.ZERO) != 0) {
                                complemento = complemento + " - JURO/MULTA";
                                Financeiro fin2 = new Financeiro();
                                fin2.setNuFin(query.getBigDecimal("NUFIN"));
                                fin2.setVlrDesdob(query.getBigDecimal("VLRJURO").add(query.getBigDecimal("VLRMULTA")));
                                fin2.setComplemento(complemento);
                                fin2.setCodCencus(query.getString("AD_CODCENCUSATBK"));
                                fin2.setCtaCtb(query.getString("AD_CONTACONTAJM"));
                                fin2.setCtaCtbPtd(query.getString("AD_CODCTACT"));
                                fin2.setRecDesp(query.getBigDecimal("RECDESP"));
                                fin2.setDt(query.getTimestamp("DHCONCILIACAO"));
                                fin2.setLancamento(query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0 ? "E"
                                                : "S");
                                listFin.add(fin2);
                        }

                        if (query.getBigDecimal("VLRDESC").compareTo(BigDecimal.ZERO) != 0) {
                                complemento = complemento + " - DESCONTO";
                                Financeiro fin3 = new Financeiro();
                                fin3.setNuFin(query.getBigDecimal("NUFIN"));
                                fin3.setVlrDesdob(query.getBigDecimal("VLRDESC"));
                                fin3.setComplemento(complemento);
                                fin3.setCodCencus(query.getString("AD_CODCENCUSATBK"));
                                fin3.setCtaCtb(query.getString("AD_CONTACONTADESC"));
                                fin3.setCtaCtbPtd(query.getString("AD_CODCTACT"));
                                fin3.setRecDesp(query.getBigDecimal("RECDESP"));
                                fin3.setDt(query.getTimestamp("DHCONCILIACAO"));
                                fin3.setLancamento(query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0 ? "E"
                                                : "S");
                                listFin.add(fin3);
                        }
                        }

                        

                        if (query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) != 0) {
                                
                        }
                }
                query.close();

                // QueryExecutor query2 = ctx.getQuery();

                // query2.setParam("DTINI", dtIni);
                // query2.setParam("DTFIN", dtFin);
                // query2.setParam("CODEMP", codEmp);

                // query2.nativeSelect("SELECT FIN.NUFIN,\r\n" + //
                // " FIN.NUMNOTA,\r\n" + //
                // " RTRIM(PAR.NOMEPARC) AS NOMEPARC,\r\n" + //
                // " IMF.VALOR,\r\n" + //
                // " FIN.RECDESP,\r\n" + //
                // " RTRIM(IMC.NOMEIMP) + ' - ' + CAST(FIN.NUMNOTA AS NVARCHAR(MAX)) + ' - ' +
                // RTRIM(PAR.NOMEPARC) AS OBSERVACAO1,\r\n"
                // + //
                // " nullValue(CAST(CAB.OBSERVACAO AS NVARCHAR(MAX)),FIN.HISTORICO) AS
                // OBSERVACAO2,\r\n"
                // + //
                // " CUS.AD_CODCENCUSATBK,\r\n" + //
                // " IAT.CTACTBSAI,\r\n" + //
                // " IAT.CTACTPTSAI,\r\n" + //
                // " MBC.DHCONCILIACAO\r\n" + //
                // "FROM TGFFIN FIN\r\n" + //
                // "INNER JOIN TGFNAT NAT\r\n" + //
                // " ON NAT.CODNAT = FIN.CODNAT\r\n" + //
                // "LEFT JOIN TGFCAB CAB\r\n" + //
                // " ON CAB.NUNOTA = FIN.NUNOTA\r\n" + //
                // "LEFT JOIN TSICUS CUS\r\n" + //
                // " ON CUS.CODCENCUS = FIN.CODCENCUS\r\n" + //
                // "INNER JOIN TGFPAR PAR\r\n" + //
                // " ON PAR.CODPARC = FIN.CODPARC\r\n" + //
                // "INNER JOIN TGFIMF IMF\r\n" + //
                // " ON IMF.NUFIN = FIN.NUFIN\r\n" + //
                // "INNER JOIN TGFIMC IMC\r\n" + //
                // " ON IMC.CODIMP = IMF.CODIMP \r\n" + //
                // "LEFT JOIN AD_IMPAUTBANK IAT\r\n" + //
                // " ON IAT.CODIMP = IMC.CODIMP\r\n" + //
                // "LEFT JOIN TCSCON CON\r\n" + //
                // " ON CON.NUMCONTRATO = CAB.NUMCONTRATO\r\n" + //
                // "WHERE onlyDate(MBC.DHCONCILIACAO) >= {DTINI}\r\n" + //
                // "AND onlyDate(MBC.DHCONCILIACAO) <= {DTFIN}\r\n" + //
                // "AND NAT.AD_CONTABILIZAVEL = 'S'\r\n" + //
                // "AND ((nullValue(CAB.NUMCONTRATO,0) = 0) OR (CAB.NUMCONTRATO <> 0 AND
                // CON.TIPO = 'L') OR (CAB.NUMCONTRATO <> 0 AND CON.TIPO = 'M'))\r\n"
                // + //
                // "AND FIN.CODEMP = {CODEMP}");

                // while (query2.next()) {
                // String complemento = query2.getString("OBSERVACAO2") != null
                // ? query2.getString("OBSERVACAO1") + " - " + query2.getString("OBSERVACAO2")
                // : query2.getString("OBSERVACAO1");
                // complemento = query2.getString("AD_CODCENCUSATBK") != null
                // ? query2.getString("AD_CODCENCUSATBK") + " - " + complemento
                // : complemento;
                // Financeiro fin = new Financeiro();
                // fin.setNuFin(query2.getBigDecimal("NUFIN"));
                // fin.setVlrDesdob(query2.getBigDecimal("VALOR"));
                // fin.setComplemento(complemento);
                // fin.setCodCencus(query2.getString("AD_CODCENCUSATBK"));
                // fin.setCtaCtb(query2.getString("CTACTBSAI"));
                // fin.setCtaCtbPtd(query2.getString("CTACTPTSAI"));
                // fin.setRecDesp(query2.getBigDecimal("RECDESP"));
                // fin.setDt(query2.getTimestamp("DHCONCIL"));
                // fin.setLancamento(query2.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE)
                // == 0 ? "E" : "S");
                // listFin.add(fin);
                // }

                // query2.close();
        }

        private void buscaFinanceiroProvisao(ContextoAcao ctx) throws Exception {
                QueryExecutor query = ctx.getQuery();
                query.setParam("DTINI", dtIni);
                query.setParam("DTFIN", dtFin);
                query.setParam("CODEMP", codEmp);

                query.nativeSelect("WITH AggregatedProd AS (\r\n" + //
                                "    SELECT CAB.NUNOTA, STRING_AGG(PRO.DESCRPROD, ', ') AS DESCRPROD\r\n" + //
                                "    FROM TGFCAB CAB\r\n" + //
                                "    INNER JOIN TGFITE ITE ON ITE.NUNOTA = CAB.NUNOTA\r\n" + //
                                "    INNER JOIN TGFPRO PRO ON ITE.CODPROD = PRO.CODPROD\r\n" + //
                                "    GROUP BY CAB.NUNOTA\r\n" + //
                                ")\r\n" + //
                                "\r\n" + //
                                "SELECT SUM(ITE.VLRTOT) AS VLRTOT,\r\n" + //
                                "       CAB.NUNOTA,\r\n" + //
                                "       FIN.NUMNOTA,\r\n" + //
                                "       RTRIM(PAR.NOMEPARC) AS NOMEPARC,\r\n" + //
                                "       FIN.RECDESP,\r\n" + //
                                "       nullValue(CAST(CAB.OBSERVACAO AS NVARCHAR(MAX)),FIN.HISTORICO) AS OBSERVACAO,\r\n"
                                + //
                                "       CUS.AD_CODCENCUSATBK,\r\n" + //
                                "       NAT.AD_CONTACONTA,\r\n" + //
                                "       NAT.AD_CONTRAPARTICA,\r\n" + //
                                "       CAB.DTNEG,\r\n" + //
                                "       AGG.DESCRPROD\r\n" + //
                                "FROM TGFCAB CAB\r\n" + //
                                "INNER JOIN TGFFIN FIN ON FIN.NUFIN = (SELECT MAX(NUFIN) FROM TGFFIN WHERE NUNOTA = CAB.NUNOTA)\r\n"
                                + //
                                "INNER JOIN TGFITE ITE ON ITE.NUNOTA = CAB.NUNOTA\r\n" + //
                                "INNER JOIN TGFNAT NAT ON NAT.CODNAT = FIN.CODNAT\r\n" + //
                                "LEFT JOIN TSICUS CUS ON CUS.CODCENCUS = FIN.CODCENCUS\r\n" + //
                                "INNER JOIN TGFPAR PAR ON PAR.CODPARC = FIN.CODPARC\r\n" + //
                                "LEFT JOIN TCSCON CON ON CON.NUMCONTRATO = CAB.NUMCONTRATO\r\n" + //
                                "LEFT JOIN AggregatedProd AGG ON CAB.NUNOTA = AGG.NUNOTA\r\n" + //
                                "WHERE onlyDate(CAB.DTNEG) >= {DTINI}\r\n" + //
                                "   AND onlyDate(CAB.DTNEG) <= {DTFIN}\r\n" + //
                                "   AND CAB.CODEMP = {CODEMP}\r\n" + //
                                "   AND NAT.AD_CONTABILIZAVEL = 'S'\r\n" + //
                                "   AND ((nullValue(CAB.NUMCONTRATO,0) = 0) OR (CAB.NUMCONTRATO <> 0 AND CON.TIPO = 'L') OR (CAB.NUMCONTRATO <> 0 AND CON.TIPO = 'M'))\r\n"
                                + //
                                "GROUP BY CAB.CODTIPVENDA,\r\n" + //
                                "          CAB.NUNOTA,\r\n" + //
                                "          FIN.NUMNOTA,\r\n" + //
                                "          RTRIM(PAR.NOMEPARC),\r\n" + //
                                "          FIN.RECDESP,\r\n" + //
                                "          nullValue(CAST(CAB.OBSERVACAO AS NVARCHAR(MAX)),FIN.HISTORICO),\r\n" + //
                                "          CUS.AD_CODCENCUSATBK,\r\n" + //
                                "          NAT.AD_CONTACONTA,\r\n" + //
                                "          NAT.AD_CONTRAPARTICA,\r\n" + //
                                "          CAB.DTNEG,\r\n" + //
                                "          AGG.DESCRPROD");

                while (query.next()) {
                        String complemento = "";
                        if (query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0) {
                                complemento = query.getString("OBSERVACAO");
                                complemento = query.getString("DESCRPROD") != null
                                                ? query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC")
                                                                + " - " + query.getString("DESCRPROD") + " - "
                                                                + complemento
                                                : query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC")
                                                                + " - " + complemento;
                                complemento = query.getString("AD_CODCENCUSATBK") != null
                                                ? query.getString("AD_CODCENCUSATBK") + " - " + complemento
                                                : complemento;
                        } else {
                                complemento = query.getString("OBSERVACAO");
                                complemento = complemento != null
                                                ? query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC")
                                                                + " - "
                                                                + complemento
                                                : query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC");
                                complemento = query.getString("AD_CODCENCUSATBK") != null
                                                ? query.getString("AD_CODCENCUSATBK") + " - " + complemento
                                                : complemento;
                        }

                        Financeiro fin = new Financeiro();
                        fin.setNuFin(query.getBigDecimal("NUNOTA"));
                        fin.setVlrDesdob(query.getBigDecimal("VLRTOT"));
                        fin.setComplemento(complemento);
                        fin.setCodCencus(query.getString("AD_CODCENCUSATBK"));
                        fin.setCtaCtb(query.getString("AD_CONTACONTA"));
                        fin.setCtaCtbPtd(query.getString("AD_CONTRAPARTICA"));
                        fin.setRecDesp(query.getBigDecimal("RECDESP"));
                        fin.setDt(query.getTimestamp("DTNEG"));
                        fin.setLancamento(query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0 ? "PE" : "PS");
                        listFin.add(fin);

                        if (query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) != 0) {
                                QueryExecutor query3 = ctx.getQuery();

                                query3.setParam("NUNOTA", query.getBigDecimal("NUNOTA"));

                                query3.nativeSelect("SELECT \n" + //
                                                "    CAB.NUNOTA,\n" + //
                                                "    ROUND(FIN.VLRDESDOB*(RAT.PERCRATEIO/100),2) AS VLR,\n" + //
                                                "    NAT.DESCRNAT AS COMPLEMENTO,\n" + //
                                                "    CUS.AD_CODCENCUSATBK,\n" + //
                                                "    RAT.CODNAT,\n" + //
                                                "    NAT.AD_CONTACONTA,\n" + //
                                                "    NAT.AD_CONTRAPARTICA,\n" + //
                                                "    FIN.RECDESP,\n" + //
                                                "    CAB.DTNEG\n" + //
                                                "FROM \n" + //
                                                "TGFCAB CAB\n" + //
                                                "INNER JOIN TGFFIN FIN ON FIN.NUFIN  = (SELECT MAX(NUFIN) FROM TGFFIN WHERE NUNOTA = CAB.NUNOTA)\n"
                                                + //
                                                "INNER JOIN TGFRAT RAT ON RAT.NUFIN = FIN.NUFIN\n" + //
                                                "INNER JOIN TGFNAT NAT ON NAT.CODNAT = RAT.CODNAT\n" + //
                                                "LEFT JOIN TSICUS CUS ON CUS.CODCENCUS = FIN.CODCENCUS\n" + //
                                                "LEFT JOIN TSICTA CTA ON CTA.CODCTABCOINT = FIN.CODCTABCOINT\n" + //
                                                "INNER JOIN TGFMBC MBC ON MBC.NUBCO = FIN.NUBCO\n" + //
                                                "WHERE CAB.NUNOTA={NUNOTA} AND RAT.ORIGEM='F'");
                                while (query3.next()) {
                                        Financeiro fin4 = new Financeiro();
                                        fin4.setNuFin(query3.getBigDecimal("NUNOTA"));
                                        fin4.setVlrDesdob(query3.getBigDecimal("VLR"));
                                        fin4.setComplemento(query3.getString("COMPLEMENTO"));
                                        fin4.setCodCencus(query3.getString("AD_CODCENCUSATBK"));
                                        fin4.setCtaCtb(query3.getString("AD_CONTACONTA"));
                                        fin4.setCtaCtbPtd(query3.getString("AD_CONTRAPARTICA"));
                                        fin4.setRecDesp(query3.getBigDecimal("RECDESP"));
                                        fin4.setDt(query3.getTimestamp("DTNEG"));
                                        fin4.setLancamento(
                                                        query3.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0
                                                                        ? "E"
                                                                        : "S");
                                        listFin.add(fin4);
                                }
                                query3.close();
                        }

                }
                query.close();

                QueryExecutor query2 = ctx.getQuery();

                query2.setParam("DTINI", dtIni);
                query2.setParam("DTFIN", dtFin);
                query2.setParam("CODEMP", codEmp);

                query2.nativeSelect("SELECT CAB.NUNOTA,\r\n" + //
                                "      (IMN.VALOR) AS VALOR,\r\n" + //
                                "      FIN.RECDESP,\r\n" + //
                                "      CUS.AD_CODCENCUSATBK,\r\n" + //
                                "      NAT.AD_CONTRAPARTICA,\r\n" + //
                                "      IAT.CTACTBFTR,\r\n" + //
                                "      IAT.CTACTBSAI,\r\n" + //
                                "      CAB.DTNEG,\r\n" + //
                                "      RTRIM(IMC.NOMEIMP) + ' - ' + CAST(FIN.NUMNOTA AS NVARCHAR(MAX)) + ' - ' + RTRIM(PAR.NOMEPARC) AS OBSERVACAO1,\r\n"
                                + //
                                "      nullValue(CAST(CAB.OBSERVACAO AS NVARCHAR(MAX)),FIN.HISTORICO) AS OBSERVACAO2,\r\n"
                                + //
                                "    (\r\n" + //
                                "        SELECT STRING_AGG(PRO.DESCRPROD, ', ') \r\n" + //
                                "        FROM TGFITE ITE\r\n" + //
                                "        JOIN TGFPRO PRO ON ITE.CODPROD = PRO.CODPROD\r\n" + //
                                "        WHERE ITE.NUNOTA = CAB.NUNOTA\r\n" + //
                                "    ) AS DESCRPROD\r\n" + //
                                " FROM TGFCAB CAB\r\n" + //
                                "INNER JOIN TGFFIN FIN ON FIN.NUFIN = (SELECT MAX(NUFIN) FROM TGFFIN WHERE NUNOTA = CAB.NUNOTA)\r\n"
                                + //
                                "INNER JOIN TGFITE ITE\r\n" + //
                                "   ON ITE.NUNOTA = CAB.NUNOTA\r\n" + //
                                "INNER JOIN TGFNAT NAT\r\n" + //
                                "   ON NAT.CODNAT = FIN.CODNAT\r\n" + //
                                " LEFT JOIN TSICUS CUS\r\n" + //
                                "   ON CUS.CODCENCUS = FIN.CODCENCUS\r\n" + //
                                "INNER JOIN TGFIMN IMN\r\n" + //
                                "   ON IMN.NUNOTA = CAB.NUNOTA\r\n" + //
                                "INNER JOIN TGFIMC IMC\r\n" + //
                                "   ON IMC.CODIMP = IMN.CODIMP\r\n" + //
                                "INNER JOIN TGFPAR PAR\r\n" + //
                                "   ON PAR.CODPARC = FIN.CODPARC\r\n" + //
                                "LEFT JOIN AD_IMPAUTBANK IAT\r\n" + //
                                "   ON IAT.CODIMP = IMC.CODIMP\r\n" + //
                                "LEFT JOIN TCSCON CON\r\n" + //
                                "   ON CON.NUMCONTRATO = CAB.NUMCONTRATO\r\n" + //
                                "WHERE onlyDate(CAB.DTNEG) >= {DTINI}\r\n" + //
                                "  AND onlyDate(CAB.DTNEG) <= {DTFIN}\r\n" + //
                                "  AND NAT.AD_CONTABILIZAVEL = 'S'\r\n" + //
                                "  AND ((nullValue(CAB.NUMCONTRATO,0) = 0) OR (CAB.NUMCONTRATO <> 0 AND CON.TIPO = 'L') OR (CAB.NUMCONTRATO <> 0 AND CON.TIPO = 'M'))\r\n"
                                + //
                                "  AND CAB.CODEMP = {CODEMP}");

                while (query2.next()) {
                        String complemento = query2.getString("DESCRPROD") != null
                                        ? query2.getString("OBSERVACAO1") + " - " + query2.getString("DESCRPROD")
                                        : query2.getString("OBSERVACAO1");
                        complemento = query2.getString("AD_CODCENCUSATBK") != null
                                        ? query2.getString("AD_CODCENCUSATBK") + " - " + complemento
                                        : complemento;
                        Financeiro fin = new Financeiro();
                        fin.setNuFin(query2.getBigDecimal("NUNOTA"));
                        fin.setVlrDesdob(query2.getBigDecimal("VALOR"));
                        fin.setComplemento(complemento);
                        fin.setCodCencus(query2.getString("AD_CODCENCUSATBK"));
                        fin.setCtaCtb(query2.getString("AD_CONTRAPARTICA"));
                        fin.setCtaCtbPtd(query2.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0
                                        ? query2.getString("CTACTBFTR")
                                        : query2.getString("CTACTBSAI"));
                        fin.setRecDesp(query2.getBigDecimal("RECDESP"));
                        fin.setDt(query2.getTimestamp("DTNEG"));
                        fin.setLancamento(query2.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0 ? "PE" : "PS");
                        listFin.add(fin);
                }

                query2.close();
        }

        private void buscaFinanceiroLiquidacaoContratos(ContextoAcao ctx) throws Exception {
                QueryExecutor query = ctx.getQuery();

                query.setParam("DTINI", dtIni);
                query.setParam("DTFIN", dtFin);
                query.setParam("CODEMP", codEmp);

                query.nativeSelect("SELECT FIN.NUFIN,\r\n" + //
                                "      CAB.NUMCONTRATO,\r\n" + //
                                "      FIN.NUMNOTA,\r\n" + //
                                "      RTRIM(PAR.NOMEPARC) AS NOMEPARC,\r\n" + //
                                "      FIN.VLRDESDOB,\r\n" + //
                                "      FIN.VLRJURO,\r\n" + //
                                "      FIN.VLRMULTA,\r\n" + //
                                "      FIN.VLRDESC,\r\n" + //
                                "      FIN.RECDESP,\r\n" + //
                                "      RTRIM(nullValue(CAST(CAB.OBSERVACAO AS VARCHAR(MAX)), FIN.HISTORICO)) AS OBSERVACAO,\r\n"
                                + //
                                "      CUS.AD_CODCENCUSATBK,\r\n" + //
                                "      CTA.AD_CODCTACT,\r\n" + //
                                "      NAT.AD_CONTRAPARTICA,\r\n" + //
                                "      CTA.AD_CODCTACT,\r\n" + //
                                "      NAT.AD_CONTACONTAJM,\r\n" + //
                                "      NAT.AD_CONTACONTADESC,\r\n" + //
                                "      MBC.DHCONCILIACAO,\r\n" + //
                                "    (\r\n" + //
                                "        SELECT STRING_AGG(PRO.DESCRPROD, ', ') \r\n" + //
                                "        FROM TGFITE ITE\r\n" + //
                                "        JOIN TGFPRO PRO ON ITE.CODPROD = PRO.CODPROD\r\n" + //
                                "        WHERE ITE.NUNOTA = CAB.NUNOTA\r\n" + //
                                "    ) AS DESCRPROD\r\n" + //
                                " FROM TGFFIN FIN\r\n" + //
                                "INNER JOIN TGFCAB CAB\r\n" + //
                                "   ON CAB.NUNOTA = FIN.NUNOTA\r\n" + //
                                "INNER JOIN TGFNAT NAT\r\n" + //
                                "   ON NAT.CODNAT = FIN.CODNAT\r\n" + //
                                "LEFT JOIN TSICUS CUS\r\n" + //
                                "   ON CUS.CODCENCUS = FIN.CODCENCUS\r\n" + //
                                "LEFT JOIN TSICTA CTA\r\n" + //
                                "   ON CTA.CODCTABCOINT = FIN.CODCTABCOINT\r\n" + //
                                "INNER JOIN TGFPAR PAR\r\n" + //
                                "   ON PAR.CODPARC = FIN.CODPARC\r\n" + //
                                "INNER JOIN TCSCON CON\r\n" + //
                                "   ON CON.NUMCONTRATO = CAB.NUMCONTRATO\r\n" + //
                                "INNER JOIN TGFMBC MBC\r\n" + //
                                "   ON MBC.NUBCO = FIN.NUBCO\r\n" + //
                                "WHERE onlyDate(MBC.DHCONCILIACAO) >= {DTINI}\r\n" + //
                                "  AND onlyDate(MBC.DHCONCILIACAO) <= {DTFIN}\r\n" + //
                                "  AND NAT.AD_CONTABILIZAVEL = 'S'\r\n" + //
                                "  AND ((CAB.NUMCONTRATO <> 0 AND CON.TIPO <> 'L') AND (CAB.NUMCONTRATO <> 0 AND CON.TIPO <> 'M'))\r\n"
                                + //
                                "  AND FIN.CODEMP = {CODEMP}");

                while (query.next()) {
                        String complemento = "";
                        if (query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0) {
                                complemento = query.getString("OBSERVACAO");
                                complemento = query.getString("DESCRPROD") != null
                                                ? query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC")
                                                                + " - " + query.getString("DESCRPROD") + " - "
                                                                + complemento
                                                : query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC")
                                                                + " - " + complemento;
                                complemento = query.getString("AD_CODCENCUSATBK") != null
                                                ? query.getString("AD_CODCENCUSATBK") + " - " + complemento
                                                : complemento;
                        } else {
                                complemento = query.getString("OBSERVACAO");
                                complemento = complemento != null
                                                ? query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC")
                                                                + " - "
                                                                + complemento
                                                : query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC");
                                complemento = query.getString("AD_CODCENCUSATBK") != null
                                                ? query.getString("AD_CODCENCUSATBK") + " - " + complemento
                                                : complemento;
                        }
                        Financeiro fin = new Financeiro();
                        fin.setNuFin(query.getBigDecimal("NUFIN"));
                        fin.setVlrDesdob(query.getBigDecimal("VLRDESDOB"));
                        fin.setComplemento(complemento);
                        fin.setCodCencus(query.getString("AD_CODCENCUSATBK"));
                        fin.setCtaCtb(query.getString("AD_CONTRAPARTICA"));
                        fin.setCtaCtbPtd(query.getString("AD_CODCTACT"));
                        fin.setRecDesp(query.getBigDecimal("RECDESP"));
                        fin.setDt(query.getTimestamp("DHCONCILIACAO"));
                        fin.setLancamento(query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0 ? "E" : "S");
                        fin.setContrato(true);
                        fin.setNumContrato(query.getBigDecimal("NUMCONTRATO"));
                        listFin.add(fin);

                        if (query.getBigDecimal("VLRJURO").compareTo(BigDecimal.ZERO) != 0
                                        || query.getBigDecimal("VLRMULTA").compareTo(BigDecimal.ZERO) != 0) {
                                complemento = complemento + " - JURO/MULTA";
                                ;
                                Financeiro fin2 = new Financeiro();
                                fin2.setNuFin(query.getBigDecimal("NUFIN"));
                                fin2.setVlrDesdob(query.getBigDecimal("VLRJURO").add(query.getBigDecimal("VLRMULTA")));
                                fin2.setComplemento(complemento);
                                fin2.setCodCencus(query.getString("AD_CODCENCUSATBK"));
                                fin2.setCtaCtb(query.getString("AD_CONTACONTAJM"));
                                fin2.setCtaCtbPtd(query.getString("AD_CODCTACT"));
                                fin2.setRecDesp(query.getBigDecimal("RECDESP"));
                                fin2.setDt(query.getTimestamp("DHCONCILIACAO"));
                                fin2.setLancamento(query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0 ? "E"
                                                : "S");
                                fin2.setContrato(true);
                                fin2.setNumContrato(query.getBigDecimal("NUMCONTRATO"));
                                listFin.add(fin2);
                        }

                        if (query.getBigDecimal("VLRDESC").compareTo(BigDecimal.ZERO) != 0) {
                                complemento = complemento + " - DESCONTO";
                                Financeiro fin3 = new Financeiro();
                                fin3.setNuFin(query.getBigDecimal("NUFIN"));
                                fin3.setVlrDesdob(query.getBigDecimal("VLRDESC"));
                                fin3.setComplemento(complemento);
                                fin3.setCodCencus(query.getString("AD_CODCENCUSATBK"));
                                fin3.setCtaCtb(query.getString("AD_CONTACONTADESC"));
                                fin3.setCtaCtbPtd(query.getString("AD_CODCTACT"));
                                fin3.setRecDesp(query.getBigDecimal("RECDESP"));
                                fin3.setDt(query.getTimestamp("DHCONCILIACAO"));
                                fin3.setLancamento(query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0 ? "E"
                                                : "S");
                                listFin.add(fin3);
                        }

                        if (query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) != 0) {
                                QueryExecutor query2 = ctx.getQuery();

                                query2.setParam("NUFIN", query.getBigDecimal("NUFIN"));

                                query2.nativeSelect("SELECT \n" + //
                                                "    RAT.NUFIN,\n" + //
                                                "    ROUND(FIN.VLRDESDOB*(RAT.PERCRATEIO/100),2) AS VLR,\n" + //
                                                "    NAT.DESCRNAT AS COMPLEMENTO,\n" + //
                                                "    CUS.AD_CODCENCUSATBK,\n" + //
                                                "    RAT.CODNAT,\n" + //
                                                "    NAT.AD_CONTRAPARTICA,\n" + //
                                                "    CTA.AD_CODCTACT,\n" + //
                                                "    FIN.RECDESP,\n" + //
                                                "    MBC.DHCONCILIACAO\n" + //
                                                "FROM TGFRAT RAT\n" + //
                                                "INNER JOIN TGFFIN FIN ON FIN.NUFIN = RAT.NUFIN\n" + //
                                                "INNER JOIN TGFNAT NAT ON NAT.CODNAT = RAT.CODNAT\n" + //
                                                "LEFT JOIN TSICUS CUS ON CUS.CODCENCUS = FIN.CODCENCUS\n" + //
                                                "LEFT JOIN TSICTA CTA ON CTA.CODCTABCOINT = FIN.CODCTABCOINT\n" + //
                                                "INNER JOIN TGFMBC MBC ON MBC.NUBCO = FIN.NUBCO\n" + //
                                                "WHERE RAT.NUFIN={NUFIN} AND RAT.ORIGEM='F'");
                                while (query2.next()) {
                                        Financeiro fin4 = new Financeiro();
                                        fin4.setNuFin(query2.getBigDecimal("NUFIN"));
                                        fin4.setVlrDesdob(query2.getBigDecimal("VLR"));
                                        fin4.setComplemento(query2.getString("COMPLEMENTO"));
                                        fin4.setCodCencus(query2.getString("AD_CODCENCUSATBK"));
                                        fin4.setCtaCtb(query2.getString("AD_CONTRAPARTICA"));
                                        fin4.setCtaCtbPtd(query2.getString("AD_CODCTACT"));
                                        fin4.setRecDesp(query2.getBigDecimal("RECDESP"));
                                        fin4.setDt(query2.getTimestamp("DHCONCILIACAO"));
                                        fin4.setLancamento(
                                                        query2.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0
                                                                        ? "E"
                                                                        : "S");
                                        listFin.add(fin4);
                                }
                                query2.close();
                        }
                }
                query.close();

                // QueryExecutor query2 = ctx.getQuery();

                // query2.setParam("DTINI", dtIni);
                // query2.setParam("DTFIN", dtFin);
                // query2.setParam("CODEMP", codEmp);

                // query2.nativeSelect("SELECT FIN.NUFIN,\r\n" + //
                // " FIN.NUMNOTA,\r\n" + //
                // " CAB.NUMCONTRATO,\r\n" + //
                // " RTRIM(PAR.NOMEPARC) AS NOMEPARC,\r\n" + //
                // " IMF.VALOR,\r\n" + //
                // " FIN.RECDESP,\r\n" + //
                // " RTRIM(IMC.NOMEIMP) + ' - ' + CAST(FIN.NUMNOTA AS NVARCHAR(MAX)) + ' - ' +
                // RTRIM(PAR.NOMEPARC) AS OBSERVACAO1,\r\n"
                // + //
                // " nullValue(CAST(CAB.OBSERVACAO AS NVARCHAR(MAX)),FIN.HISTORICO) AS
                // OBSERVACAO2,\r\n"
                // + //
                // " CUS.AD_CODCENCUSATBK,\r\n" + //
                // " IAT.CTACTBCTR,\r\n" + //
                // " IAT.CTACTPTCTR,\r\n" + //
                // " MBC.DHCONCILIACAO\r\n" + //
                // "FROM TGFFIN FIN\r\n" + //
                // "INNER JOIN TGFNAT NAT\r\n" + //
                // " ON NAT.CODNAT = FIN.CODNAT\r\n" + //
                // "INNER JOIN TGFCAB CAB\r\n" + //
                // " ON CAB.NUNOTA = FIN.NUNOTA\r\n" + //
                // "LEFT JOIN TSICUS CUS\r\n" + //
                // " ON CUS.CODCENCUS = FIN.CODCENCUS\r\n" + //
                // "INNER JOIN TGFPAR PAR\r\n" + //
                // " ON PAR.CODPARC = FIN.CODPARC\r\n" + //
                // "INNER JOIN TGFIMF IMF\r\n" + //
                // " ON IMF.NUFIN = FIN.NUFIN\r\n" + //
                // "INNER JOIN TGFIMC IMC\r\n" + //
                // " ON IMC.CODIMP = IMF.CODIMP \r\n" + //
                // "LEFT JOIN AD_IMPAUTBANK IAT\r\n" + //
                // " ON IAT.CODIMP = IMC.CODIMP\r\n" + //
                // "INNER JOIN TCSCON CON\r\n" + //
                // " ON CON.NUMCONTRATO = CAB.NUMCONTRATO\r\n" + //
                // "WHERE onlyDate(MBC.DHCONCILIACAO) >= {DTINI}\r\n" + //
                // "AND onlyDate(MBC.DHCONCILIACAO) <= {DTFIN}\r\n" + //
                // "AND NAT.AD_CONTABILIZAVEL = 'S'\r\n" + //
                // "AND ((CAB.NUMCONTRATO <> 0 AND CON.TIPO <> 'L') AND (CAB.NUMCONTRATO <> 0
                // AND CON.TIPO <> 'M'))\r\n"
                // + //
                // "AND FIN.CODEMP = {CODEMP}");

                // while (query2.next()) {
                // String complemento = query2.getString("OBSERVACAO2") != null
                // ? query2.getString("OBSERVACAO1") + " - " + query2.getString("OBSERVACAO2")
                // : query2.getString("OBSERVACAO1");
                // complemento = query2.getString("AD_CODCENCUSATBK") != null
                // ? query2.getString("AD_CODCENCUSATBK") + " - " + complemento
                // : complemento;
                // Financeiro fin = new Financeiro();
                // fin.setNuFin(query2.getBigDecimal("NUFIN"));
                // fin.setVlrDesdob(query2.getBigDecimal("VALOR"));
                // fin.setComplemento(complemento);
                // fin.setCodCencus(query2.getString("AD_CODCENCUSATBK"));
                // fin.setCtaCtb(query2.getString("CTACTBCTR"));
                // fin.setCtaCtbPtd(query2.getString("CTACTPTCTR"));
                // fin.setRecDesp(query2.getBigDecimal("RECDESP"));
                // fin.setDt(query2.getTimestamp("DHCONCIL"));
                // fin.setLancamento(query2.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE)
                // == 0 ? "E" : "S");
                // fin.setContrato(true);
                // fin.setNumContrato(query2.getBigDecimal("NUMCONTRATO"));
                // listFin.add(fin);
                // }

                // query2.close();
        }

        private void buscaFinanceiroProvisaoContratos(ContextoAcao ctx) throws Exception {
                QueryExecutor query = ctx.getQuery();
                query.setParam("DTINI", dtIni);
                query.setParam("DTFIN", dtFin);
                query.setParam("CODEMP", codEmp);

                query.nativeSelect("WITH AggregatedProd AS (\r\n" + //
                                "    SELECT CAB.NUNOTA, STRING_AGG(PRO.DESCRPROD, ', ') AS DESCRPROD\r\n" + //
                                "    FROM TGFCAB CAB\r\n" + //
                                "    INNER JOIN TGFITE ITE ON ITE.NUNOTA = CAB.NUNOTA\r\n" + //
                                "    INNER JOIN TGFPRO PRO ON ITE.CODPROD = PRO.CODPROD\r\n" + //
                                "    GROUP BY CAB.NUNOTA\r\n" + //
                                ")\r\n" + //
                                "\r\n" + //
                                "SELECT SUM(ITE.VLRTOT) AS VLRTOT,\r\n" + //
                                "       CAB.NUNOTA,\r\n" + //
                                "       FIN.NUMNOTA,\r\n" + //
                                "       RTRIM(PAR.NOMEPARC) AS NOMEPARC,\r\n" + //
                                "       FIN.RECDESP,\r\n" + //
                                "       nullValue(CAST(CAB.OBSERVACAO AS NVARCHAR(MAX)),FIN.HISTORICO) AS OBSERVACAO,\r\n"
                                + //
                                "       CUS.AD_CODCENCUSATBK,\r\n" + //
                                "       NAT.AD_CONTACONTADIF,\r\n" + //
                                "       NAT.AD_CONTRAPARTICA,\r\n" + //
                                "       CAB.DTNEG,\r\n" + //
                                "       CAB.NUMCONTRATO,\r\n" + //
                                "       AGG.DESCRPROD\r\n" + //
                                "FROM TGFCAB CAB\r\n" + //
                                "INNER JOIN TGFFIN FIN ON FIN.NUFIN = (SELECT MAX(NUFIN) FROM TGFFIN WHERE NUNOTA = CAB.NUNOTA)\r\n"
                                + //
                                "INNER JOIN TGFITE ITE ON ITE.NUNOTA = CAB.NUNOTA\r\n" + //
                                "INNER JOIN TGFNAT NAT ON NAT.CODNAT = FIN.CODNAT\r\n" + //
                                "LEFT JOIN TSICUS CUS ON CUS.CODCENCUS = FIN.CODCENCUS\r\n" + //
                                "INNER JOIN TGFPAR PAR ON PAR.CODPARC = FIN.CODPARC\r\n" + //
                                "LEFT JOIN TCSCON CON ON CON.NUMCONTRATO = CAB.NUMCONTRATO   \r\n" + //
                                "LEFT JOIN AggregatedProd AGG ON CAB.NUNOTA = AGG.NUNOTA\r\n" + //
                                "WHERE onlyDate(CAB.DTNEG) >= {DTINI}\r\n" + //
                                "   AND onlyDate(CAB.DTNEG) <= {DTFIN}\r\n" + //
                                "   AND CAB.CODEMP = {CODEMP}\r\n" + //
                                "   AND NAT.AD_CONTABILIZAVEL = 'S'\r\n" + //
                                "  AND ((CAB.NUMCONTRATO <> 0 AND CON.TIPO <> 'L') AND (CAB.NUMCONTRATO <> 0 AND CON.TIPO <> 'M'))\r\n"
                                + //
                                "GROUP BY CAB.CODTIPVENDA,\r\n" + //
                                "          CAB.NUNOTA,\r\n" + //
                                "          FIN.NUMNOTA,\r\n" + //
                                "          RTRIM(PAR.NOMEPARC),\r\n" + //
                                "          FIN.RECDESP,\r\n" + //
                                "          nullValue(CAST(CAB.OBSERVACAO AS NVARCHAR(MAX)),FIN.HISTORICO),\r\n" + //
                                "          CUS.AD_CODCENCUSATBK,\r\n" + //
                                "          NAT.AD_CONTACONTADIF,\r\n" + //
                                "          NAT.AD_CONTRAPARTICA,\r\n" + //
                                "          CAB.DTNEG,\r\n" + //
                                "          CAB.NUMCONTRATO,\r\n" + //
                                "          AGG.DESCRPROD");

                while (query.next()) {
                        String complemento = "";
                        if (query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0) {
                                complemento = query.getString("OBSERVACAO");
                                complemento = query.getString("DESCRPROD") != null
                                                ? query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC")
                                                                + " - " + query.getString("DESCRPROD") + " - "
                                                                + complemento
                                                : query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC")
                                                                + " - " + complemento;
                                complemento = query.getString("AD_CODCENCUSATBK") != null
                                                ? query.getString("AD_CODCENCUSATBK") + " - " + complemento
                                                : complemento;
                        } else {
                                complemento = query.getString("OBSERVACAO");
                                complemento = complemento != null
                                                ? query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC")
                                                                + " - "
                                                                + complemento
                                                : query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC");
                                complemento = query.getString("AD_CODCENCUSATBK") != null
                                                ? query.getString("AD_CODCENCUSATBK") + " - " + complemento
                                                : complemento;
                        }
                        Financeiro fin = new Financeiro();
                        fin.setNuFin(query.getBigDecimal("NUNOTA"));
                        fin.setVlrDesdob(query.getBigDecimal("VLRTOT"));
                        fin.setComplemento(complemento);
                        fin.setCodCencus(query.getString("AD_CODCENCUSATBK"));
                        fin.setCtaCtb(query.getString("AD_CONTACONTADIF"));
                        fin.setCtaCtbPtd(query.getString("AD_CONTRAPARTICA"));
                        fin.setRecDesp(query.getBigDecimal("RECDESP"));
                        fin.setDt(query.getTimestamp("DTNEG"));
                        fin.setLancamento(query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0 ? "PE" : "PS");
                        fin.setContrato(true);
                        fin.setNumContrato(query.getBigDecimal("NUMCONTRATO"));
                        listFin.add(fin);

                        if (query.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) != 0) {
                                QueryExecutor query3 = ctx.getQuery();

                                query3.setParam("NUNOTA", query.getBigDecimal("NUNOTA"));

                                query3.nativeSelect("SELECT \n" + //
                                                "    CAB.NUNOTA,\n" + //
                                                "    ROUND(FIN.VLRDESDOB*(RAT.PERCRATEIO/100),2) AS VLR,\n" + //
                                                "    NAT.DESCRNAT AS COMPLEMENTO,\n" + //
                                                "    CUS.AD_CODCENCUSATBK,\n" + //
                                                "    RAT.CODNAT,\n" + //
                                                "    NAT.AD_CONTACONTA,\n" + //
                                                "    NAT.AD_CONTRAPARTICA,\n" + //
                                                "    FIN.RECDESP,\n" + //
                                                "    CAB.DTNEG\n" + //
                                                "FROM \n" + //
                                                "TGFCAB CAB\n" + //
                                                "INNER JOIN TGFFIN FIN ON FIN.NUFIN  = (SELECT MAX(NUFIN) FROM TGFFIN WHERE NUNOTA = CAB.NUNOTA)\n"
                                                + //
                                                "INNER JOIN TGFRAT RAT ON RAT.NUFIN = FIN.NUFIN\n" + //
                                                "INNER JOIN TGFNAT NAT ON NAT.CODNAT = RAT.CODNAT\n" + //
                                                "LEFT JOIN TSICUS CUS ON CUS.CODCENCUS = FIN.CODCENCUS\n" + //
                                                "LEFT JOIN TSICTA CTA ON CTA.CODCTABCOINT = FIN.CODCTABCOINT\n" + //
                                                "INNER JOIN TGFMBC MBC ON MBC.NUBCO = FIN.NUBCO\n" + //
                                                "WHERE CAB.NUNOTA={NUNOTA} AND RAT.ORIGEM='F'");
                                while (query3.next()) {
                                        Financeiro fin4 = new Financeiro();
                                        fin4.setNuFin(query3.getBigDecimal("NUNOTA"));
                                        fin4.setVlrDesdob(query3.getBigDecimal("VLR"));
                                        fin4.setComplemento(query3.getString("COMPLEMENTO"));
                                        fin4.setCodCencus(query3.getString("AD_CODCENCUSATBK"));
                                        fin4.setCtaCtb(query3.getString("AD_CONTACONTA"));
                                        fin4.setCtaCtbPtd(query3.getString("AD_CONTRAPARTICA"));
                                        fin4.setRecDesp(query3.getBigDecimal("RECDESP"));
                                        fin4.setDt(query3.getTimestamp("DTNEG"));
                                        fin4.setLancamento(
                                                        query3.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0
                                                                        ? "E"
                                                                        : "S");
                                        listFin.add(fin4);
                                }
                                query3.close();
                        }

                }
                query.close();

                QueryExecutor query2 = ctx.getQuery();

                query2.setParam("DTINI", dtIni);
                query2.setParam("DTFIN", dtFin);
                query2.setParam("CODEMP", codEmp);

                query2.nativeSelect("SELECT CAB.NUNOTA,\r\n" + //
                                "      (IMN.VALOR) AS VALOR,\r\n" + //
                                "      FIN.RECDESP,\r\n" + //
                                "      CUS.AD_CODCENCUSATBK,\r\n" + //
                                "      NAT.AD_CONTRAPARTICA,\r\n" + //
                                "      IAT.CTACTBFTR,\r\n" + //
                                "      IAT.CTACTBSAI,\r\n" + //
                                "      CAB.DTNEG,\r\n" + //
                                "      CAB.NUMCONTRATO,\r\n" + //
                                "      RTRIM(IMC.NOMEIMP) + ' - ' + CAST(FIN.NUMNOTA AS NVARCHAR(MAX)) + ' - ' + RTRIM(PAR.NOMEPARC) AS OBSERVACAO1,\r\n"
                                + //
                                "      nullValue(CAST(CAB.OBSERVACAO AS NVARCHAR(MAX)),FIN.HISTORICO) AS OBSERVACAO2,\r\n"
                                + //
                                "    (\r\n" + //
                                "        SELECT STRING_AGG(PRO.DESCRPROD, ', ') \r\n" + //
                                "        FROM TGFITE ITE\r\n" + //
                                "        JOIN TGFPRO PRO ON ITE.CODPROD = PRO.CODPROD\r\n" + //
                                "        WHERE ITE.NUNOTA = CAB.NUNOTA\r\n" + //
                                "    ) AS DESCRPROD\r\n" + //
                                " FROM TGFCAB CAB\r\n" + //
                                "INNER JOIN TGFFIN FIN ON FIN.NUFIN = (SELECT MAX(NUFIN) FROM TGFFIN WHERE NUNOTA = CAB.NUNOTA)\r\n"
                                + //
                                "INNER JOIN TGFITE ITE\r\n" + //
                                "   ON ITE.NUNOTA = CAB.NUNOTA\r\n" + //
                                "INNER JOIN TGFNAT NAT\r\n" + //
                                "   ON NAT.CODNAT = FIN.CODNAT\r\n" + //
                                " LEFT JOIN TSICUS CUS\r\n" + //
                                "   ON CUS.CODCENCUS = FIN.CODCENCUS\r\n" + //
                                "INNER JOIN TGFIMN IMN\r\n" + //
                                "   ON IMN.NUNOTA = CAB.NUNOTA\r\n" + //
                                "INNER JOIN TGFIMC IMC\r\n" + //
                                "   ON IMC.CODIMP = IMN.CODIMP\r\n" + //
                                "INNER JOIN TGFPAR PAR\r\n" + //
                                "   ON PAR.CODPARC = FIN.CODPARC\r\n" + //
                                "LEFT JOIN AD_IMPAUTBANK IAT\r\n" + //
                                "   ON IAT.CODIMP = IMC.CODIMP\r\n" + //
                                "LEFT JOIN TCSCON CON\r\n" + //
                                "   ON CON.NUMCONTRATO = CAB.NUMCONTRATO\r\n" + //
                                "WHERE onlyDate(CAB.DTNEG) >= {DTINI}\r\n" + //
                                "  AND onlyDate(CAB.DTNEG) <= {DTFIN}\r\n" + //
                                "  AND NAT.AD_CONTABILIZAVEL = 'S'\r\n" + //
                                "  AND ((CAB.NUMCONTRATO <> 0 AND CON.TIPO <> 'L') AND (CAB.NUMCONTRATO <> 0 AND CON.TIPO <> 'M'))\r\n"
                                + //
                                "  AND CAB.CODEMP = {CODEMP}");

                while (query2.next()) {
                        String complemento = query2.getString("DESCRPROD") != null
                                        ? query2.getString("OBSERVACAO1") + " - " + query2.getString("DESCRPROD")
                                        : query2.getString("OBSERVACAO1");
                        complemento = query2.getString("AD_CODCENCUSATBK") != null
                                        ? query2.getString("AD_CODCENCUSATBK") + " - " + complemento
                                        : complemento;
                        Financeiro fin = new Financeiro();
                        fin.setNuFin(query2.getBigDecimal("NUNOTA"));
                        fin.setVlrDesdob(query2.getBigDecimal("VALOR"));
                        fin.setComplemento(complemento);
                        fin.setCodCencus(query2.getString("AD_CODCENCUSATBK"));
                        fin.setCtaCtb(query2.getString("AD_CONTRAPARTICA"));
                        fin.setCtaCtbPtd(query2.getString("CTACTBFTR"));
                        fin.setRecDesp(query2.getBigDecimal("RECDESP"));
                        fin.setDt(query2.getTimestamp("DTNEG"));
                        fin.setLancamento(query2.getBigDecimal("RECDESP").compareTo(BigDecimal.ONE) == 0 ? "PE" : "PS");
                        fin.setContrato(true);
                        fin.setNumContrato(query2.getBigDecimal("NUMCONTRATO"));
                        listFin.add(fin);
                }

                query2.close();
        }

        private void buscaDiferimentoContratos(ContextoAcao ctx) throws Exception {
                QueryExecutor query = ctx.getQuery();
                query.setParam("DTINI", dtIni);
                query.setParam("DTFIN", dtFin);
                query.setParam("CODEMP", codEmp);

                query.nativeSelect("SELECT\r\n" + //
                                "CON.NUMCONTRATO,\r\n" + //
                                "FORMAT(CAB.DTNEG, 'dd/MM/yy') AS DTCONTRATO,\r\n" + //
                                "CON.TIPO,\r\n" + //
                                "(SELECT SUM(VLRTOT) FROM TGFITE WHERE NUNOTA=CAB.NUNOTA) AS VLRTOT,\r\n" + //
                                "FIN.NUMNOTA,\r\n" + //
                                "FIN.NUFIN,\r\n" + //
                                "FIN.RECDESP,\r\n" + //
                                "RTRIM(nullValue(CAST(CAB.OBSERVACAO AS VARCHAR(MAX)), FIN.HISTORICO)) AS OBSERVACAO,\r\n"
                                + //
                                "RTRIM(PAR.NOMEPARC) AS NOMEPARC,\r\n" + //
                                "NAT.AD_CONTACONTADIF,\r\n" + //
                                "CUS.AD_CODCENCUSATBK,\r\n" + //
                                "NAT.AD_CONTACONTA,\r\n" + //
                                "    (\r\n" + //
                                "        SELECT STRING_AGG(PRO.DESCRPROD, ', ') \r\n" + //
                                "        FROM TGFITE ITE\r\n" + //
                                "        JOIN TGFPRO PRO ON ITE.CODPROD = PRO.CODPROD\r\n" + //
                                "        WHERE ITE.NUNOTA = CAB.NUNOTA\r\n" + //
                                "    ) AS DESCRPROD\r\n" + //
                                "FROM TCSCON CON\r\n" + //
                                "INNER JOIN TGFCAB CAB\r\n" + //
                                "    ON CAB.NUMCONTRATO = CON.NUMCONTRATO\r\n" + //
                                "    AND CAB.NUNOTA = (SELECT MAX(NUNOTA) FROM TGFCAB WHERE NUMCONTRATO = CON.NUMCONTRATO)\r\n"
                                + //
                                "INNER JOIN TGFFIN FIN\r\n" + //
                                "    ON FIN.NUFIN = (SELECT MAX(NUFIN) FROM TGFFIN WHERE NUNOTA = CAB.NUNOTA)\r\n" + //
                                "INNER JOIN TGFNAT NAT\r\n" + //
                                "    ON NAT.CODNAT = FIN.CODNAT\r\n" + //
                                "INNER JOIN TGFPAR PAR\r\n" + //
                                "   ON PAR.CODPARC = FIN.CODPARC\r\n" + //
                                "LEFT JOIN TSICUS CUS\r\n" + //
                                "    ON CUS.CODCENCUS = FIN.CODCENCUS\r\n" + //
                                "WHERE CON.ATIVO = 'S' \r\n" + //
                                "    AND CON.TIPO NOT IN ('M', 'L') \r\n" + //
                                "    AND CON.CODEMP = {CODEMP}\r\n" + //
                                "    AND CAB.DTNEG BETWEEN {DTINI} AND {DTFIN}\r\n" + //
                                "UNION ALL\r\n" + //
                                "SELECT\r\n" + //
                                "CON.NUMCONTRATO,\r\n" + //
                                "FORMAT(DATEADD(DAY, -DAY({DTINI}) + 1, {DTINI}), 'dd/MM/yy') AS DTCONTRATO,\r\n" + //
                                "CON.TIPO,\r\n" + //
                                "(SELECT SUM(VLRTOT) FROM TGFITE WHERE NUNOTA=CAB.NUNOTA) AS VLRTOT,\r\n" + //
                                "FIN.NUMNOTA,\r\n" + //
                                "FIN.NUFIN,\r\n" + //
                                "FIN.RECDESP,\r\n" + //
                                "RTRIM(nullValue(CAST(CAB.OBSERVACAO AS VARCHAR(MAX)), FIN.HISTORICO)) AS OBSERVACAO,\r\n"
                                + //
                                "RTRIM(PAR.NOMEPARC) AS NOMEPARC,\r\n" + //
                                "NAT.AD_CONTACONTADIF,\r\n" + //
                                "CUS.AD_CODCENCUSATBK,\r\n" + //
                                "NAT.AD_CONTACONTA,\r\n" + //
                                "    (\r\n" + //
                                "        SELECT STRING_AGG(PRO.DESCRPROD, ', ') \r\n" + //
                                "        FROM TGFITE ITE\r\n" + //
                                "        JOIN TGFPRO PRO ON ITE.CODPROD = PRO.CODPROD\r\n" + //
                                "        WHERE ITE.NUNOTA = CAB.NUNOTA\r\n" + //
                                "    ) AS DESCRPROD\r\n" + //
                                "FROM TCSCON CON\r\n" + //
                                "INNER JOIN TGFCAB CAB\r\n" + //
                                "    ON CAB.NUMCONTRATO = CON.NUMCONTRATO\r\n" + //
                                "    AND CAB.NUNOTA = (SELECT MAX(NUNOTA) FROM TGFCAB WHERE NUMCONTRATO = CON.NUMCONTRATO)\r\n"
                                + //
                                "INNER JOIN TGFFIN FIN\r\n" + //
                                "    ON FIN.NUFIN = (SELECT MAX(NUFIN) FROM TGFFIN WHERE NUNOTA = CAB.NUNOTA)\r\n" + //
                                "INNER JOIN TGFNAT NAT\r\n" + //
                                "    ON NAT.CODNAT = FIN.CODNAT\r\n" + //
                                "INNER JOIN TGFPAR PAR\r\n" + //
                                "   ON PAR.CODPARC = FIN.CODPARC\r\n" + //
                                "LEFT JOIN TSICUS CUS\r\n" + //
                                "    ON CUS.CODCENCUS = FIN.CODCENCUS\r\n" + //
                                "WHERE CON.ATIVO = 'S' \r\n" + //
                                "    AND CON.TIPO NOT IN ('M', 'L') \r\n" + //
                                "    AND CON.CODEMP = {CODEMP}\r\n" + //
                                "    AND DAY({DTINI}) = 1\r\n" + //
                                "    AND MONTH(CAB.DTNEG) <> MONTH({DTINI})\r\n" + //
                                "    AND (SELECT COUNT(1)/2 FROM AD_CONLANCAUTBANK WHERE NUFIN=FIN.NUFIN) < \r\n" + //
                                "    (CASE\r\n" + //
                                "    WHEN TIPO = 'A' THEN 12 \r\n" + //
                                "    WHEN TIPO = 'B' THEN 2\r\n" + //
                                "    WHEN TIPO = 'Q' THEN 4\r\n" + //
                                "    WHEN TIPO = 'S' THEN 6\r\n" + //
                                "    WHEN TIPO = 'T' THEN 3\r\n" + //
                                "    ELSE 0 \r\n" + //
                                "    END)");

                while (query.next()) {
                        String complemento = query.getString("OBSERVACAO");
                        complemento = query.getString("DESCRPROD") != null
                                        ? query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC")
                                                        + " - " + query.getString("DESCRPROD") + " - " + complemento
                                        : query.getString("NUMNOTA") + " - " + query.getString("NOMEPARC") + " - "
                                                        + complemento;
                        complemento = query.getString("AD_CODCENCUSATBK") != null
                                        ? query.getString("AD_CODCENCUSATBK") + " - " + complemento
                                        : complemento;

                        Financeiro fin = new Financeiro();
                        fin.setNuFin(query.getBigDecimal("NUFIN"));
                        fin.setVlrDesdob(query.getBigDecimal("VLRTOT"));
                        fin.setComplemento(complemento);
                        fin.setCodCencus(query.getString("AD_CODCENCUSATBK"));
                        fin.setCtaCtb(query.getString("AD_CONTACONTADIF"));
                        fin.setCtaCtbPtd(query.getString("AD_CONTACONTA"));
                        fin.setRecDesp(query.getBigDecimal("RECDESP"));
                        fin.setDt(TimeUtils.toTimestamp(query.getString("DTCONTRATO"), "dd/MM/yy"));
                        fin.setLancamento("D");
                        fin.setContrato(true);
                        fin.setNumContrato(query.getBigDecimal("NUMCONTRATO"));
                        fin.setPeriodoContrato(query.getString("TIPO"));
                        listFin.add(fin);

                }
                query.close();
        }

}

```