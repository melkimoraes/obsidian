evento de solicitacao de coleta → cliente,qtd, id SAC.

Evento na RNC → Mateus vai criar → Coleta TERCEIRO.

Coleta de Terceiro somente nessa tela.

Aprovação Diretoria → ultima.

---
Campo no SAC:
tipo da coleta → frota/terceiro
pedido → lote
quantidade reclamada →
quantidade palete → 
peso →
nota da coleta já emitida → dois tipos
advertência da coleta → dois tipos
coleta frota → ver com o thiago valor e a tabela em sankhya. Pra preencher custo evento.
Campo Calculado → Data Coleta → com base na informação que eles colocarem NA TELA DA COLETA.

FAZER TRIGGER → EVENTO TIPO 15

E-mail automatico na inserção evento → coleta solicitada.


Frota:
Cliente
Cliente coleta
Cidade
E campos que foram preenchidos no SAC e ID
se tipo frota → habilitar campo pra ajudante
qtd ajudante
valor do ajudante
Veiculo - placa carreta
Motorista
Tipo do caminhao (ja tem no cadastro do veiculo)
Data Coleta

Terceiros:
Cliente
Cliente coleta
Cidade
E campos que foram preenchidos no SAC e ID
Data Coleta

Aba de Cotação Coleta → aprovação sobe as informações da transportadora vencedora
Valor orçamento
Transportadora
Veiculo - Placa carreta
Motorista (cpf/rg/cnh/celular)?

Status/Fluxos
Iniciada → Fretes (Thiago Cintra/ Matheus Laveli/ Marcos Silva) → depois que o Thiago colocou os orçamentos - > finalizado cotação.
Aguard. Envio Cotação →  Paloma → envia todas as cotações via mensageria? o retorno da mensageria a Paloma aprovaria a transportadora venceroda.
Aguard. Aprovação Cotação →
Cotação Aprovada →  STatus 2 → aguardando preenchimento frete. Thiago preencheu a placa → finaliza preenchimento coleta
Aguardando Coleta → Ver com o tiago temos algum status/rotina do sankhya que a coleta chegou, finalizou? O tiago informar o ticket que entrou a coleta?
Coleta Finalizada → 
Liberação Diretoria → enviar alerta para todos (CSC/SAC/Fretes/Financeiro) vai para o finaceiro.
Aguard. Financeiro → Adriana → finalizou (pago)


Iniciada → Transporte (Renato/Mateus/Juan/Gustavo/Zildiomar) → finaliza informações
Aguard. Envio Cliente → SAC envia informaçoes pro cliente
Aguardando Coleta → ver oq vai finalizar a coleta, alguem informar o ticket que entrou a coleta?
Finalizada Coleta



Renato → ver custo da frota.

---

create or replace TRIGGER TRG_IU_PROSACEVE_15_ART 
BEFORE INSERT OR UPDATE OR DELETE ON AD_PROSACEVE
FOR EACH ROW
WHEN (NEW.STATUS = 15)
DECLARE
V_COUNT     NUMBER;
V_CODEMP    NUMBER; 
V_CODPARC   NUMBER;
BEGIN
  IF INSERTING OR UPDATING THEN
    IF :NEW.TIPOCOLETA IS NULL THEN
        STP_MSGERRO_ART('Ação Cancelada!','Evento de coleta exige preencher o campo Tipo Coleta!');
    END IF;
    IF :NEW.QTDRECLAMADA IS NULL THEN
        STP_MSGERRO_ART('Ação Cancelada!','Evento de coleta exige preencher o campo Qtd. Reclamada!');
    END IF;
    IF :NEW.QTDPALETE IS NULL THEN
        STP_MSGERRO_ART('Ação Cancelada!','Evento de coleta exige preencher o campo Qtd. Palete!');
    END IF;
    IF :NEW.PESO IS NULL THEN
        STP_MSGERRO_ART('Ação Cancelada!','Evento de coleta exige preencher o campo Peso!');
    END IF;
    IF :NEW.PEDIDOCOLETA IS NULL THEN
        STP_MSGERRO_ART('Ação Cancelada!','Evento de coleta exige preencher o campo Pedido!');
    END IF;
    
    SELECT COUNT(1)
    INTO V_COUNT
    FROM AD_COL WHERE SACRNC = :NEW.ID;
    
    IF V_COUNT <> 0 THEN
        STP_MSGERRO_ART('Ação Cancelada!','Já existe coleta gerada pra esse evento!','Cancelar Coleta se necessário alteração');
    END IF;
    
    SELECT CODEMP, CODPARC
    INTO V_CODEMP, V_CODPARC
    FROM AD_PROSAC
    WHERE ID = :NEW.ID;
    
    INSERT INTO AD_COL (IDCOL, CODEMP, TIPOCOLETA, CLIENTE, QTDRECLAMADA, QTDPALETE, PESO, PEDIDOCOLETA, SACRNC, STATUS)
    VALUES
    ((SELECT NVL(MAX(IDCOL),0)+1 FROM AD_COL), V_CODEMP, :NEW.TIPOCOLETA, V_CODPARC, :NEW.QTDRECLAMADA, :NEW.QTDPALETE, :NEW.PESO, :NEW.PEDIDOCOLETA, :NEW.ID, 'I');
  END IF;
  
  IF DELETING THEN
    SELECT COUNT(1)
    INTO V_COUNT
    FROM AD_COL WHERE SACRNC = :OLD.ID;
    
    IF V_COUNT <> 0 THEN
        STP_MSGERRO_ART('Ação Cancelada!','Já existe coleta gerada pra esse evento!','Cancelar Coleta se necessário deleção');
    END IF;
  END IF;
END;