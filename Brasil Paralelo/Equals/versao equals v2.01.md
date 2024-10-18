package br.com.sankhya.gm.integracaoequalsv2.schedule;  
  
import br.com.sankhya.extensions.actionbutton.AcaoRotinaJava;  
import br.com.sankhya.extensions.actionbutton.ContextoAcao;  
import br.com.sankhya.gm.integracaoequalsv2.model.Adquirente;  
import br.com.sankhya.gm.integracaoequalsv2.model.ResponseAjuste;  
import br.com.sankhya.gm.integracaoequalsv2.model.ResponseVenda;  
import br.com.sankhya.gm.integracaoequalsv2.util.HttpUtils;  
import br.com.sankhya.jape.EntityFacade;  
import br.com.sankhya.jape.core.JapeSession;  
import br.com.sankhya.jape.dao.JdbcWrapper;  
import br.com.sankhya.jape.sql.NativeSql;  
import br.com.sankhya.jape.vo.DynamicVO;  
import br.com.sankhya.jape.vo.EntityVO;  
import br.com.sankhya.jape.wrapper.JapeFactory;  
import br.com.sankhya.modelcore.auth.AuthenticationInfo;  
import br.com.sankhya.modelcore.util.EntityFacadeFactory;  
import br.com.sankhya.modelcore.util.SPBeanUtils;  
import br.com.sankhya.ws.ServiceContext;  
import com.google.gson.Gson;  
import com.google.gson.JsonObject;  
import com.google.gson.JsonParser;  
import com.google.gson.reflect.TypeToken;  
import com.sankhya.util.StringUtils;  
import org.cuckoo.core.ScheduledAction;  
import org.cuckoo.core.ScheduledActionContext;  
  
import java.lang.reflect.Type;  
import java.math.BigDecimal;  
import java.nio.charset.StandardCharsets;  
import java.sql.ResultSet;  
import java.sql.Timestamp;  
import java.time.Duration;  
import java.time.LocalDateTime;  
import java.util.ArrayList;  
import java.util.HashMap;  
import java.util.List;  
import java.util.Map;  
  
/**  
 * @author melki  
 */public class ImportaFinanceiros implements ScheduledAction, AcaoRotinaJava {  
  
    private EntityFacade dwfFacade = null;  
    private JdbcWrapper jdbc = null;  
    private static final String PREFERENCIA_TOKEN = "EQUALSAPITOKEN";  
    private static final String URL_API_AJUSTE_DEBITO_CREDITO = "https://app.equals.com.br/api/ajustesadebitocredito";  
    private static final String URL_API_RESUMO_VENDAS_GERAR_RELATORIO = "https://app.equals.com.br/api/v2/resumovendas/gerar-relatorio";  
    private static final String URL_API_RESUMO_VENDAS_CARREGAR_RELATORIO = "https://app.equals.com.br/api/v2/resumovendas/carregar-relatorio";  
    private static final String CONTENT_TYPE_JSON = "application/json";  
    String token = "";  
    String id = "";  
  
    Timestamp dt;  
  
    @Override  
    public void onTime(ScheduledActionContext sac) {  
        JapeSession.SessionHandle hnd = null;  
        ServiceContext sctx = null;  
        sctx = new ServiceContext(null);  
        sctx.setAutentication(AuthenticationInfo.getCurrent());  
        sctx.makeCurrent();  
  
        try {  
            SPBeanUtils.setupContext(sctx);  
        } catch (Exception e) {  
            e.printStackTrace();  
            sac.info("Error: Não foi Possivel Executar a Chamada SPBeanUtils.setupContext \n" + e.getMessage());  
        }  
  
        try {  
            System.out.println("Inicio do Importar Integração Equals");  
  
            dwfFacade = EntityFacadeFactory.getDWFFacade();  
            jdbc = dwfFacade.getJdbcWrapper();  
            jdbc.openSession();  
  
            LocalDateTime timestamp = LocalDateTime.now();  
            dt = Timestamp.valueOf(timestamp.minus(Duration.ofDays(1)));  
  
            DynamicVO intVO = JapeFactory.dao("AD_INTEQUALSV2").findByPK(dt);  
            if (intVO == null) {  
                token = getToken();  
  
                if (token.isEmpty()) {  
                    throw new Error("Token não encontrado, verificar preferencia " + PREFERENCIA_TOKEN);  
                }  
  
                importarFinanceiros();  
  
            }  
            System.out.println("Fim do Importar Integração Equals");  
        } catch (Exception e) {  
            System.out.println("Erro rotina da Integração Equals " + e.getMessage());  
            e.printStackTrace();  
        } finally {  
            JdbcWrapper.closeSession(jdbc);  
            jdbc.closeSession();  
        }  
    }  
  
    @Override  
    public void doAction(ContextoAcao contextoAcao) throws Exception {  
        try {  
            System.out.println("Inicio do Importar Integração Equals");  
  
            dwfFacade = EntityFacadeFactory.getDWFFacade();  
            jdbc = dwfFacade.getJdbcWrapper();  
            jdbc.openSession();  
  
            LocalDateTime timestamp = LocalDateTime.now();  
            dt = Timestamp.valueOf(timestamp.minus(Duration.ofDays(1)));  
  
            DynamicVO intVO = JapeFactory.dao("AD_INTEQUALSV2").findByPK(dt);  
            if (intVO == null) {  
                token = getToken();  
  
                if (token.isEmpty()) {  
                    throw new Error("Token não encontrado, verificar preferencia " + PREFERENCIA_TOKEN);  
                }  
  
                importarFinanceiros();  
  
            }  
            System.out.println("Fim do Importar Integração Equals");  
            contextoAcao.setMensagemRetorno("Financeiros Importados!");  
        } catch (Exception e) {  
            System.out.println("Erro rotina da Integração Equals " + e.getMessage());  
            e.printStackTrace();  
            throw e;  
        } finally {  
            JdbcWrapper.closeSession(jdbc);  
            jdbc.closeSession();  
        }  
    }  
  
    private void importarFinanceiros() throws Exception {  
  
        ArrayList<Adquirente> listAdquirentesVendas = resumoVendas(dt);  
        ArrayList<Adquirente> listAdquirentesAjuste = ajusteDebitoCredito(dt);  
  
        salvarIntegracaoFinanceira(dt);  
  
        salvarAdquirentes(listAdquirentesVendas, "AD_INTEQUALSV2VND");  
        salvarAdquirentes(listAdquirentesAjuste, "AD_INTEQUALSV2AJT");  
    }  
  
    private void salvarIntegracaoFinanceira(Timestamp dt) throws Exception {  
        DynamicVO intEqualsVO = (DynamicVO) dwfFacade.getDefaultValueObjectInstance("AD_INTEQUALSV2");  
        intEqualsVO.setProperty("DT", dt);  
        intEqualsVO.setProperty("IDRELATORIO", id);  
        dwfFacade.createEntity("AD_INTEQUALSV2", (EntityVO) intEqualsVO);  
    }  
  
    private void salvarAdquirentes(List<Adquirente> adquirentes, String tabela) throws Exception {  
        for (Adquirente adquirente : adquirentes) {  
            DynamicVO intEqualsVO = (DynamicVO) dwfFacade.getDefaultValueObjectInstance(tabela);  
            intEqualsVO.setProperty("DT", dt);  
            intEqualsVO.setProperty("JSON", adquirente.getJson().toCharArray());  
            dwfFacade.createEntity(tabela, (EntityVO) intEqualsVO);  
        }  
    }  
  
    private ArrayList<Adquirente> ajusteDebitoCredito(Timestamp dt) throws Exception {  
        ArrayList<Adquirente> listAdquirentes = new ArrayList<>();  
  
        Map<String, String> headers = new HashMap<>();  
        headers.put("Content-Type", CONTENT_TYPE_JSON);  
        headers.put("Authorization", "Basic " + token);  
  
        JsonObject requestBody = new JsonObject();  
        requestBody.addProperty("dataInicial", StringUtils.formatTimestamp(dt, "yyyy-MM-dd"));  
        requestBody.addProperty("dataFinal", StringUtils.formatTimestamp(dt, "yyyy-MM-dd"));  
        requestBody.addProperty("opcaoData", "O");  
        requestBody.addProperty("exibicao", "A");  
  
        String response = HttpUtils.sendPostRequest(URL_API_AJUSTE_DEBITO_CREDITO, headers, requestBody.toString());  
  
        byte[] bytesUtf8 = response.getBytes();  
        response = new String(bytesUtf8, StandardCharsets.UTF_8);  
  
        System.out.println(response);  
  
        Type listType = new TypeToken<List<ResponseAjuste.Item>>() {  
        }.getType();  
  
        List<ResponseAjuste.Item> items = new Gson().fromJson(response, listType);  
  
        for (ResponseAjuste.Item item : items) {  
            Adquirente ad = new Adquirente();  
            ad.setValor(BigDecimal.valueOf(item.valorAjuste));  
            ad.setId(item.idAdquirente);  
            ad.setMotivoAjuste(item.motivoAjuste);  
            ad.addJson(new Gson().toJson(item));  
            listAdquirentes.add(ad);  
        }  
  
        return listAdquirentes;  
    }  
  
    private ArrayList<Adquirente> resumoVendas(Timestamp dt) throws Exception {  
        id = gerarResumoVendas(dt);  
  
        ResponseVenda response = lerResumoVendas(id, "1");  
  
        ArrayList<Adquirente> listAdquirentes = new ArrayList<>();  
  
        agruparItems(listAdquirentes, response.items);  
  
        int totalItems = response.currentPage * response.pageSize;  
  
        //loop caso ultrapasse os 1000 registros da primeira pagina!  
        while (response.totalItems > totalItems) {  
  
            int nextPage = response.currentPage + 1;  
            response = lerResumoVendas(id, Integer.toString(nextPage));  
  
            agruparItems(listAdquirentes, response.items);  
  
            totalItems = response.currentPage * response.pageSize;  
  
        }  
        return listAdquirentes;  
    }  
  
    private void agruparItems(ArrayList<Adquirente> listAdquirentes, ArrayList<ResponseVenda.Item> items) {  
        for (ResponseVenda.Item item : items) {  
            // Verificando se o adquirente já está na lista de adquirentes  
            boolean adquirenteExistente = false;  
            for (Adquirente adquirente : listAdquirentes) {  
                if (adquirente.getId() == item.adquirente.id) {  
                    adquirenteExistente = true;  
  
                    // Adicionando o valor de comissão ao adquirente  
                    adquirente.addComissao(BigDecimal.valueOf(item.valorComissao));  
  
                    // Concatenando o JSON do item ao adquirente  
                    adquirente.addJson(item.toJson());  
                    break;  
                }  
            }  
  
            // Se o adquirente não existir na lista, criamos um novo adquirente  
            if (!adquirenteExistente) {  
                Adquirente novoAdquirente = new Adquirente();  
                novoAdquirente.setId(item.adquirente.id);  
                novoAdquirente.setValor(BigDecimal.valueOf(item.valorComissao));  
  
                // Concatenando o JSON do item ao adquirente  
                novoAdquirente.addJson(item.toJson());  
  
                // Adicionando o novo adquirente à lista  
                listAdquirentes.add(novoAdquirente);  
            }  
        }  
    }  
  
    private String gerarResumoVendas(Timestamp dt) throws Exception {  
        Map<String, String> headers = new HashMap<>();  
        headers.put("Content-Type", CONTENT_TYPE_JSON);  
        headers.put("Authorization", "Basic " + token);  
  
        JsonObject requestBody = new JsonObject();  
        requestBody.addProperty("dataInicial", StringUtils.formatTimestamp(dt, "yyyy-MM-dd"));  
        requestBody.addProperty("dataFinal", StringUtils.formatTimestamp(dt, "yyyy-MM-dd"));  
  
        String response = HttpUtils.sendPostRequest(URL_API_RESUMO_VENDAS_GERAR_RELATORIO, headers, requestBody.toString());  
        JsonObject responseJson = new JsonParser().parse(response).getAsJsonObject();  
  
        System.out.println("gerarResumoVendas=" + response);  
  
        return responseJson.get("idRelatorio").getAsString();  
    }  
  
    private ResponseVenda lerResumoVendas(String id, String page) throws Exception {  
        Map<String, String> headers = new HashMap<>();  
        headers.put("Content-Type", CONTENT_TYPE_JSON);  
        headers.put("Authorization", "Basic " + token);  
  
        Map<String, String> params = new HashMap<>();  
        params.put("idRelatorio", id);  
        params.put("page", page);  
  
        String response = "";  
        boolean processando = true;  
        while (processando) {  
            Thread.sleep(10000);  
            try {  
                response = HttpUtils.sendGetRequest(URL_API_RESUMO_VENDAS_CARREGAR_RELATORIO, headers, params, null);  
                processando = false;  
            } catch (Exception e) {  
                e.printStackTrace();  
            }  
        }  
  
        JsonObject responseJson = new JsonParser().parse(response).getAsJsonObject();  
  
        System.out.println(response);  
  
        Gson gson = new Gson();  
        return gson.fromJson(responseJson, ResponseVenda.class);  
    }  
  
    private String getToken() throws Exception {  
        try {  
            String texto = "";  
            String sql = "SELECT TEXTO FROM TSIPAR WHERE CHAVE = :CHAVE ";  
  
            NativeSql sqlBusca = new NativeSql(jdbc);  
            sqlBusca.setReuseStatements(false);  
            sqlBusca.setNamedParameter("CHAVE", PREFERENCIA_TOKEN);  
            sqlBusca.appendSql(sql);  
  
            ResultSet result = sqlBusca.executeQuery(sql);  
  
            if (result.next()) {  
                if (result.getString("TEXTO") != null) {  
                    texto = result.getString("TEXTO");  
                }  
            }  
            return texto;  
        } catch (Exception e) {  
            System.out.println("br.com.sankhya.gm.integracaoequals.schedule.ImportaFinanceiros.getToken()" + e.getMessage());  
            throw e;  
        }  
    }  
}