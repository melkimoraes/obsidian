```

package br.com.sankhya.gm.integracaoguru.schedule;

import java.math.BigDecimal;
import java.math.RoundingMode;
import java.sql.Date;
import java.sql.ResultSet;
import java.sql.Timestamp;
import java.text.Normalizer;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Collection;
import java.util.regex.Pattern;

import br.com.sankhya.extensions.actionbutton.AcaoRotinaJava;
import br.com.sankhya.extensions.actionbutton.ContextoAcao;
import br.com.sankhya.extensions.actionbutton.Registro;
import br.com.sankhya.gm.integracaoguru.helper.NotaHelper;
import br.com.sankhya.gm.integracaoguru.model.ItemNota;
import br.com.sankhya.gm.integracaoguru.model.Nota;
import br.com.sankhya.gm.integracaoguru.model.Pedido;
import br.com.sankhya.gm.integracaoguru.model.Pedido.Contact;
import br.com.sankhya.gm.integracaoguru.parse.ParsePedido;
import br.com.sankhya.gm.integracaoguru.service.CidadeCodeService;
import br.com.sankhya.jape.EntityFacade;
import br.com.sankhya.jape.bmp.PersistentLocalEntity;
import br.com.sankhya.jape.core.JapeSession;
import br.com.sankhya.jape.sql.NativeSql;
import br.com.sankhya.jape.util.FinderWrapper;
import br.com.sankhya.jape.vo.DynamicVO;
import br.com.sankhya.jape.vo.EntityVO;
import br.com.sankhya.jape.wrapper.JapeFactory;
import br.com.sankhya.jape.wrapper.JapeWrapper;
import br.com.sankhya.modelcore.util.DynamicEntityNames;
import br.com.sankhya.modelcore.util.EntityFacadeFactory;

public class GerarNotasGuru implements AcaoRotinaJava {

    @Override
    public void doAction(ContextoAcao ca) throws Exception {
        EntityFacade dwfFacade = EntityFacadeFactory.getDWFFacade();

        Registro[] linhas = ca.getLinhas();

        if (linhas.length == 0) {
            ca.mostraErro("Nenhuma linha selecionada na grade.\r\n Ação cancelada.");
        }

        for (Registro registro : linhas) {
            BigDecimal id = new BigDecimal(registro.getCampo("ID").toString());
            FinderWrapper finderPag = new FinderWrapper("AD_INTGURUPAGAPRV", "this.ID = ?", new Object[] { id });
            finderPag.setMaxResults(1000);
            Collection<PersistentLocalEntity> colPag = dwfFacade.findByDynamicFinder(finderPag);

            for (PersistentLocalEntity pagPLE : colPag) {
                DynamicVO pagVO = (DynamicVO) pagPLE.getValueObject();

                FinderWrapper finder = new FinderWrapper("AD_INTGURUPEDAPRV", " this.ID = ? AND this.PAGINA = ?",
                        new Object[] { id, pagVO.asBigDecimal("PAGINA") });
                Collection<DynamicVO> colRegs = dwfFacade.findByDynamicFinderAsVO(finder);

                for (DynamicVO regVO : colRegs) {
                    Nota nota = new Nota();
                    try {
                        nota.setCodEmp(BigDecimal.ONE);
                        Pedido responsePedido = ParsePedido.fromJson(regVO.asString("JSON"));

                        int timeCreatedUnix = responsePedido.dates.created_at;
                        int timeConfirmedUnix = responsePedido.dates.confirmed_at;
                        Date data = new Date(timeCreatedUnix * 1000L);

                        SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy");
                        nota.setDtNeg(sdf.format(data));
                        data = new Date(timeConfirmedUnix * 1000L);
                        nota.setDtEntSai(sdf.format(data));

                        DynamicVO configGuruVO = getConfigGuru(dwfFacade);
                        if (configGuruVO == null) {
                            throw new Exception("Configurações Guru não encontrado.");
                        }

                        DynamicVO parcVO = getParceiro(dwfFacade, responsePedido.getContact());
                        nota.setCodParc(parcVO.asBigDecimal("CODPARC"));

                        DynamicVO produtoGuruVO = validaProdutoGuru(dwfFacade,
                                new BigDecimal(responsePedido.getProduct().id));

                        if (produtoGuruVO == null) {
                            throw new Exception("ID do Produto não encontrado, ID " + responsePedido.getProduct().id);
                        }
                        if (produtoGuruVO.asBigDecimal("CODTIPVENDA") == null) {
                            throw new Exception(
                                    "ID Produto " + responsePedido.getProduct().id + ", sem Tipo de negociação.");
                        }
                        nota.setCodTipNeg(produtoGuruVO.asBigDecimal("CODTIPVENDA"));

                        BigDecimal vlr = new BigDecimal(responsePedido.getPayment().total);
                        if ((produtoGuruVO.asBigDecimal("EMITNFE") != null
                                || produtoGuruVO.asBigDecimal("EMITNFE").compareTo(BigDecimal.ZERO) == 1)
                                && !existeNFEId(id, pagVO.asBigDecimal("PAGINA"), regVO.asString("IDPED"), dwfFacade)) {
                            ItemNota item = new ItemNota();

                            if (produtoGuruVO.asBigDecimal("CODPROD") == null) {
                                throw new Exception(
                                        "ID Produto " + responsePedido.getProduct().id + ", sem Produto para NFe.");
                            }

                            item.setCodProd(produtoGuruVO.asBigDecimal("CODPROD"));
                            item.setVlrUnit(vlr.divide(
                                    produtoGuruVO.asBigDecimal("QTDFATUR"),
                                    2, RoundingMode.HALF_UP)
                                    .multiply(produtoGuruVO.asBigDecimal("EMITNFE").divide(new BigDecimal("100"),
                                            2, RoundingMode.HALF_UP)));

                            for (int i = 1; i <= produtoGuruVO.asBigDecimal("QTDFATUR").intValue(); i++) {
                                Timestamp dtFatur = null;
                                if (i == 1) {
                                    Calendar calendar = Calendar.getInstance();

                                    calendar.setTime(data);

                                    calendar.add(Calendar.DAY_OF_MONTH,
                                            produtoGuruVO.asBigDecimal("QTDDIASEMISSAO").intValue());

                                    Date novaData = new Date(calendar.getTimeInMillis());
                                    dtFatur = new Timestamp(novaData.getTime());
                                } else {
                                    Calendar calendar = Calendar.getInstance();

                                    calendar.setTime(data);

                                    calendar.add(Calendar.MONTH, i - 1);

                                    Date novaData = new Date(calendar.getTimeInMillis());
                                    dtFatur = new Timestamp(novaData.getTime());
                                }
                                NotaHelper notaHelper = new NotaHelper(nota, item,
                                        configGuruVO.asBigDecimal("CODTIPOPERNFE"),regVO.asString("IDPED"));
                                BigDecimal nuNota = notaHelper.gerarNota();
                                insereAbaNfe(id, pagVO.asBigDecimal("PAGINA"), regVO.asString("IDPED"), nuNota, i,
                                        dtFatur);
                            }
                        }
                        if ((produtoGuruVO.asBigDecimal("EMITNFSE") != null
                                || produtoGuruVO.asBigDecimal("EMITNFSE").compareTo(BigDecimal.ZERO) == 1)
                                && !existeNFSEId(id, pagVO.asBigDecimal("PAGINA"), regVO.asString("IDPED"),
                                        dwfFacade)) {
                            ItemNota item = new ItemNota();

                            if (produtoGuruVO.asBigDecimal("CODSERVICO") == null) {
                                throw new Exception(
                                        "ID Produto " + responsePedido.getProduct().id + ", sem Serviço para NFSe.");
                            }

                            item.setCodProd(produtoGuruVO.asBigDecimal("CODSERVICO"));
                            item.setVlrUnit(vlr.divide(
                                    produtoGuruVO.asBigDecimal("QTDFATUR"),
                                    2, RoundingMode.HALF_UP)
                                    .multiply(produtoGuruVO.asBigDecimal("EMITNFSE").divide(new BigDecimal("100"),
                                            2, RoundingMode.HALF_UP)));
                            for (int i = 1; i <= produtoGuruVO.asBigDecimal("QTDFATUR").intValue(); i++) {
                                Timestamp dtFatur = null;
                                if (i == 1) {
                                    Calendar calendar = Calendar.getInstance();

                                    calendar.setTime(data);

                                    calendar.add(Calendar.DAY_OF_MONTH,
                                            produtoGuruVO.asBigDecimal("QTDDIASEMISSAO").intValue());

                                    Date novaData = new Date(calendar.getTimeInMillis());
                                    dtFatur = new Timestamp(novaData.getTime());
                                } else {
                                    Calendar calendar = Calendar.getInstance();

                                    calendar.setTime(data);

                                    calendar.add(Calendar.MONTH, i - 1);

                                    Date novaData = new Date(calendar.getTimeInMillis());
                                    dtFatur = new Timestamp(novaData.getTime());
                                }
                                NotaHelper notaHelper = new NotaHelper(nota, item,
                                        configGuruVO.asBigDecimal("CODTIPOPERNFSE"), regVO.asString("IDPED"));
                                BigDecimal nuNota = notaHelper.gerarNota();

                                insereAbaNfse(id, pagVO.asBigDecimal("PAGINA"), regVO.asString("IDPED"), nuNota, i,
                                        dtFatur);
                            }
                        }

                        resetarLog(regVO);
                    } catch (Exception e) {
                        inserirLog(regVO, e.getMessage());
                    }
                }

            }

        }
        ca.setMensagemRetorno("Pré-Notas geradas com sucesso!");
    }

    private boolean existeNFEId(BigDecimal id, BigDecimal pagina, String idProd, EntityFacade dwfFacade)
            throws Exception {
        NativeSql nativeSql = new NativeSql(dwfFacade.getJdbcWrapper());
        nativeSql.setNamedParameter("IDPED", idProd);
        nativeSql.setNamedParameter("PAGINA", pagina);
        nativeSql.setNamedParameter("ID", id);
        String query = ("SELECT COUNT(1) AS QTD FROM AD_INTGURUAPRVNFE WHERE ID = :ID AND PAGINA = :PAGINA AND IDPED = :IDPED");
        ResultSet rs = nativeSql.executeQuery(query);
        rs.next();
        if (rs.getBigDecimal("QTD").compareTo(BigDecimal.ZERO) == 0)
            return false;
        else
            return true;
    }

    private boolean existeNFSEId(BigDecimal id, BigDecimal pagina, String idProd, EntityFacade dwfFacade)
            throws Exception {
        NativeSql nativeSql = new NativeSql(dwfFacade.getJdbcWrapper());
        nativeSql.setNamedParameter("IDPED", idProd);
        nativeSql.setNamedParameter("PAGINA", pagina);
        nativeSql.setNamedParameter("ID", id);
        String query = ("SELECT COUNT(1) AS QTD FROM AD_INTGURUAPRVNFSE WHERE ID = :ID AND PAGINA = :PAGINA AND IDPED = :IDPED");
        ResultSet rs = nativeSql.executeQuery(query);
        rs.next();
        if (rs.getBigDecimal("QTD").compareTo(BigDecimal.ZERO) == 0)
            return false;
        else
            return true;
    }

    private void insereAbaNfse(BigDecimal id, BigDecimal pagina, String idProd, BigDecimal nuNota, int nroFatura,
            Timestamp dtFatur) throws Exception {
        JapeSession.SessionHandle hnd = null;
        try {
            hnd = JapeSession.open();
            hnd.execWithTX((JapeSession.TXBlock) new JapeSession.TXBlock() {
                @Override
                public void doWithTx() throws Exception {
                    EntityFacade dwf = EntityFacadeFactory.getDWFFacade();
                    DynamicVO intGuruNfeVO = (DynamicVO) dwf.getDefaultValueObjectInstance("AD_INTGURUAPRVNFSE");
                    intGuruNfeVO.setProperty("ID", id);
                    intGuruNfeVO.setProperty("PAGINA", pagina);
                    intGuruNfeVO.setProperty("IDPED", idProd);
                    intGuruNfeVO.setProperty("NUNOTA", nuNota);
                    intGuruNfeVO.setProperty("NROFATURA", new BigDecimal(nroFatura));
                    intGuruNfeVO.setProperty("DTFATUR", dtFatur);
                    dwf.createEntity("AD_INTGURUAPRVNFSE", (EntityVO) intGuruNfeVO);
                }
            });
        } catch (Exception e) {
            System.out.println("br.com.sankhya.gm.integracaoguru.GerarNotasGuru.insereAbaNfse()" + e.getMessage());
            throw e;
        } finally {
            JapeSession.close(hnd);
        }
    }

    private void insereAbaNfe(BigDecimal id, BigDecimal pagina, String idProd, BigDecimal nuNota, int nroFatura,
            Timestamp dtFatur)
            throws Exception {
        JapeSession.SessionHandle hnd = null;
        try {
            hnd = JapeSession.open();
            hnd.execWithTX((JapeSession.TXBlock) new JapeSession.TXBlock() {
                @Override
                public void doWithTx() throws Exception {
                    EntityFacade dwf = EntityFacadeFactory.getDWFFacade();
                    DynamicVO intGuruNfeVO = (DynamicVO) dwf.getDefaultValueObjectInstance("AD_INTGURUAPRVNFE");
                    intGuruNfeVO.setProperty("ID", id);
                    intGuruNfeVO.setProperty("PAGINA", pagina);
                    intGuruNfeVO.setProperty("IDPED", idProd);
                    intGuruNfeVO.setProperty("NUNOTA", nuNota);
                    intGuruNfeVO.setProperty("NROFATURA", new BigDecimal(nroFatura));
                    intGuruNfeVO.setProperty("DTFATUR", dtFatur);
                    dwf.createEntity("AD_INTGURUAPRVNFE", (EntityVO) intGuruNfeVO);
                }
            });
        } catch (Exception e) {
            System.out.println("br.com.sankhya.gm.integracaoguru.GerarNotasGuru.insereAbaNfe()" + e.getMessage());
            throw e;
        } finally {
            JapeSession.close(hnd);
        }
    }

    private DynamicVO getConfigGuru(EntityFacade dwfFacade) throws Exception {
        DynamicVO configVO = null;
        JapeWrapper configDAO = JapeFactory.dao("AD_CONFIGINTGURU");
        configVO = configDAO.findOne("this.ID = 1", null);
        return configVO;
    }

    private DynamicVO getParceiro(EntityFacade dwfFacade, Contact contato) throws Exception {
        JapeWrapper parcDAO = JapeFactory.dao(DynamicEntityNames.PARCEIRO);
        DynamicVO parcVO = parcDAO.findOne("this.CGC_CPF = ?",
                new Object[] { contato.doc });
        if (parcVO == null) {
            parcVO = cadastrarParceiro(dwfFacade, contato);
        }
        return parcVO;
    }

    private DynamicVO validaProdutoGuru(EntityFacade dwfFacade, BigDecimal idProd) throws Exception {
        DynamicVO produtoVO = null;
        JapeWrapper produtoDAO = JapeFactory.dao("AD_PRODGURU");
        produtoVO = produtoDAO.findOne("this.IDPROD = ? ", new Object[] { idProd });
        return produtoVO;
    }

    private void inserirLog(DynamicVO regVO, String message) throws Exception {
        JapeSession.SessionHandle hnd = null;
        try {
            hnd = JapeSession.open();
            hnd.execWithTX((JapeSession.TXBlock) new JapeSession.TXBlock() {
                @Override
                public void doWithTx() throws Exception {
                    JapeWrapper intGuruDAO = JapeFactory.dao("AD_INTGURUPEDAPRV");
                    intGuruDAO.prepareToUpdate(regVO)
                            .set("LOG", message)
                            .update();
                }
            });
        } catch (Exception e) {
            System.out.println("br.com.sankhya.gm.integracaoguru.GerarNotasGuru.inserirLog()" + e.getMessage());
            throw e;
        } finally {
            JapeSession.close(hnd);
        }
    }

    private void resetarLog(DynamicVO regVO) throws Exception {
        JapeSession.SessionHandle hnd = null;
        try {
            hnd = JapeSession.open();
            hnd.execWithTX((JapeSession.TXBlock) new JapeSession.TXBlock() {
                @Override
                public void doWithTx() throws Exception {
                    JapeWrapper intGuruDAO = JapeFactory.dao("AD_INTGURUPEDAPRV");
                    intGuruDAO.prepareToUpdate(regVO)
                            .set("LOG", "")
                            .update();
                }
            });
        } catch (Exception e) {
            System.out.println("br.com.sankhya.gm.integracaoguru.GerarNotasGuru.resetarLog()" + e.getMessage());
            throw e;
        } finally {
            JapeSession.close(hnd);
        }
    }

    private DynamicVO cadastrarParceiro(EntityFacade dwfFacade, Contact contact) throws Exception {
        DynamicVO parcVO = null;
        JapeSession.SessionHandle hnd = null;
        try {
            final DynamicVO[] parcVOHolder = new DynamicVO[1];
            hnd = JapeSession.open();
            hnd.execWithTX((JapeSession.TXBlock) new JapeSession.TXBlock() {
                @Override
                public void doWithTx() throws Exception {
                    BigDecimal codCid = BigDecimal.ZERO;
                    BigDecimal codUF = (BigDecimal) NativeSql.getColumnValue("CODUF", "TSIUFS", "UF LIKE ?",
                            new Object[] { contact.address_state }, dwfFacade.getJdbcWrapper());
                            System.out.println("codUF="+codUF);
                    codCid = CidadeCodeService.getCodCid(contact.address_zip_code, removerAcentos(contact.address_city), codUF,
                            dwfFacade);
                    DynamicVO newParcVO = (DynamicVO) dwfFacade
                            .getDefaultValueObjectInstance(DynamicEntityNames.PARCEIRO);
                    newParcVO.setProperty("NOMEPARC", contact.name);
                    newParcVO.setProperty("CGC_CPF", contact.doc);
                    newParcVO.setProperty("CEP", contact.address_zip_code);
                    newParcVO.setProperty("TIPPESSOA", "F");
                    newParcVO.setProperty("CODCID", codCid);
                    newParcVO.setProperty("CLIENTE", "S");
                    newParcVO.setProperty("AD_INTGURU", "S");
                    dwfFacade.createEntity(DynamicEntityNames.PARCEIRO, (EntityVO) newParcVO);
                    parcVOHolder[0] = newParcVO;
                }
            });
            parcVO = parcVOHolder[0];
        } catch (Exception e) {
            System.out.println("br.com.sankhya.gm.action.ImportarNotaExcel.insereLogImportacao()" + e.getMessage());
            throw e;
        } finally {
            JapeSession.close(hnd);
        }
        return parcVO;
    }

    public static String removerAcentos(String texto) {
        // Normaliza o texto para o formato NFD
        String textoNormalizado = Normalizer.normalize(texto, Normalizer.Form.NFD);
        
        // Usa uma expressão regular para remover os caracteres acentuados
        Pattern pattern = Pattern.compile("\\p{InCombiningDiacriticalMarks}+");
        return pattern.matcher(textoNormalizado).replaceAll("");
    }

}

```