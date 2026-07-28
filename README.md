1.ReportService
  
  /**
     * Method Name : getHeaders
     * <p>
     * Description : This method connects to the database and get specific headers via merchant
     * </p>
     *
     * @param reportName
     * @param mId
     * @param defaultHeaders List<String> - The headers for the report.
     */
    public List<String> getHeaders(String reportName,String mId , List<String> defaultHeaders) {
        log.info("fetching report headers for reportName ={} , merchantId={}", reportName, mId);
        Optional<ReportHeaderConfig> config = reportHeaderConfigRepository.findBymIdAndReportNameAndIsActive(mId, reportName, true);
        if (config.isEmpty()) {
            log.info("using default headers configuration");
            return defaultHeaders;
        }
        try {
            return objectMapper.readValue(config.get().getHeaderJson(), new TypeReference<List<String>>() {
            });
        } catch (JsonProcessingException e) {
            log.error("Error while fetching header configuration");
            throw new ReportingException(INVALID_ERROR_CODE, INVALID_HEADER_CONFIGURATION);
        }
    }

}



2.

case TRANSACTION ->
                        buildReport(reportManagementDto, getHeaders(Report.TRANSACTION.name(), reportManagementDto.getMId(), transactionHeader), reportDao.getTransaction(reportManagementDto));



3.


    private static final List<String> transactionHeader = List.of("Transaction Request Date & Time", "Transaction Success Date & Time", "Merchant Order No", "SBIEPAY ORDER ID", "Cust Id", "ATRN", "Gateway Trace Number", "Pay Mode Code", "Gateway Name", "Pay Proc", "Transaction Currency", "Merchant Order Amount", "Gateway Posting Amount", "Commission", "GST", "Order Status", "Transaction Status", "Settlement Status", "Refund Status", "Chargeback Status", "Amount Refunded", "Amount Chargeback", "CIN Number", "Merchant Other Details", "Settlement Date");


4.table

--liquibase formatted sql
--changeset REPORT:234

CREATE TABLE REPORT_HEADER_CONFIG
(
    ID RAW(16) DEFAULT SYS_GUID(),
    MID VARCHAR2(20 BYTE) NOT NULL,
    REPORT_NAME VARCHAR2(100 CHAR),
    HEADER_JSON CLOB NOT NULL,
    IS_ACTIVE NUMBER(1) NOT NULL,
    CREATED_DATE NUMBER NOT NULL ,
    CREATED_BY VARCHAR2(100 BYTE) NOT NULL ,
    UPDATED_AT NUMBER ,
    UPDATED_BY VARCHAR2(100 BYTE)
)

INSERT INTO REPORT_HEADER_CONFIG (MID,REPORT_NAME,HEADER_JSON,IS_ACTIVE,CREATED_DATE,CREATED_BY,UPDATED_AT,UPDATED_BY) VALUES
    (1003772,'TRANSACTION',
     '[
     "Transaction Request Date And Time",
     "Transaction Success Date And Time",
     "Merchant Order No",
     "SBIEPAY ORDER ID",
     "Cust Id",
     "ATRN",
     "Gateway Trace Number",
     "Pay Mode Code",
     "Gateway Name",
     "Pay Proc",
     "Transaction Currency",
     "Merchant Order Amount",
     "Gateway Posting Amount",
     "Commission",
     "GST",
     "Order Status",
     "Transaction Status",
     "Settlement Status",
     "Refund Status",
     "Chargeback Status",
     "Amount Refunded",
     "Amount Chargeback",
     "CIN Number",
     "Merchant Other Details",
     "Settlement Date"
      ]',
     1,
     (SELECT ROUND(((TRUNC(SYSTIMESTAMP,'MI')-DATE '1970-01-01') * 24 * 60 * 60) + EXTRACT (SECOND FROM SYSTIMESTAMP), 3)*1000 FROM DUAL),
     'SBIePAY', NULL, NULL
    );
