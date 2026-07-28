
public List<String> getHeaders(String reportName, String mId, Map<Integer, String> defaultHeaders) {

    log.info("fetching report headers for reportName={}, merchantId={}", reportName, mId);

    Optional<ReportHeaderConfig> config =
            reportHeaderConfigRepository.findBymIdAndReportNameAndIsActive(
                    mId,
                    reportName,
                    true);

    if (config.isEmpty()) {
        log.info("using default headers configuration");

        return defaultHeaders.entrySet()
                .stream()
                .sorted(Map.Entry.comparingByKey())
                .map(Map.Entry::getValue)
                .toList();
    }

    try {

        Map<Integer, String> headerMap =
                objectMapper.readValue(
                        config.get().getHeaderJson(),
                        new TypeReference<Map<Integer, String>>() {
                        });

        return headerMap.entrySet()
                .stream()
                .sorted(Map.Entry.comparingByKey())
                .map(Map.Entry::getValue)
                .toList();

    } catch (JsonProcessingException e) {

        log.error("Error while fetching header configuration", e);

        throw new ReportingException(
                INVALID_ERROR_CODE,
                INVALID_HEADER_CONFIGURATION);
    }
}



Ho. He part-wise copy-paste sathi.


---

Part 1 – Imports

import java.util.Map;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.core.JsonProcessingException;


---

Part 2 – transactionHeader

private static final Map<Integer, String> transactionHeader = Map.ofEntries(
        Map.entry(1, "Transaction Request Date And Time"),
        Map.entry(2, "Transaction Success Date And Time"),
        Map.entry(3, "Merchant Order No"),
        Map.entry(4, "SBIEPAY ORDER ID"),
        Map.entry(5, "Cust Id"),
        Map.entry(6, "ATRN"),
        Map.entry(7, "Gateway Trace Number"),
        Map.entry(8, "Pay Mode Code"),
        Map.entry(9, "Gateway Name"),
        Map.entry(10, "Pay Proc"),
        Map.entry(11, "Transaction Currency"),
        Map.entry(12, "Merchant Order Amount"),
        Map.entry(13, "Gateway Posting Amount"),
        Map.entry(14, "Commission"),
        Map.entry(15, "GST"),
        Map.entry(16, "Order Status"),
        Map.entry(17, "Transaction Status"),
        Map.entry(18, "Settlement Status"),
        Map.entry(19, "Refund Status"),
        Map.entry(20, "Chargeback Status"),
        Map.entry(21, "Amount Refunded"),
        Map.entry(22, "Amount Chargeback"),
        Map.entry(23, "CIN Number"),
        Map.entry(24, "Merchant Other Details"),
        Map.entry(25, "Settlement Date")
);


---

Part 3 – getHeaders()

public List<String> getHeaders(String reportName, String mId, Map<Integer, String> defaultHeaders) {

    log.info("fetching report headers for reportName ={}, merchantId={}", reportName, mId);

    Optional<ReportHeaderConfig> config =
            reportHeaderConfigRepository.findBymIdAndReportNameAndIsActive(mId, reportName, true);

    if (config.isEmpty()) {
        log.info("using default headers configuration");

        return defaultHeaders.entrySet()
                .stream()
                .sorted(Map.Entry.comparingByKey())
                .map(Map.Entry::getValue)
                .toList();
    }

    try {

        Map<String, String> headerMap =
                objectMapper.readValue(
                        config.get().getHeaderJson(),
                        new TypeReference<Map<String, String>>() {
                        });

        return headerMap.entrySet()
                .stream()
                .sorted((e1, e2) ->
                        Integer.compare(
                                Integer.parseInt(e1.getKey()),
                                Integer.parseInt(e2.getKey())
                        ))
                .map(Map.Entry::getValue)
                .toList();

    } catch (JsonProcessingException e) {

        log.error("Error while fetching header configuration", e);

        throw new ReportingException(
                INVALID_ERROR_CODE,
                INVALID_HEADER_CONFIGURATION
        );
    }
}


---

Part 4 – buildReport()

case TRANSACTION ->
        buildReport(
                reportManagementDto,
                getHeaders(
                        Report.TRANSACTION.name(),
                        reportManagementDto.getMId(),
                        transactionHeader
                ),
                reportDao.getTransaction(reportManagementDto)
        );


---

Part 5 – Liquibase SQL

HEADER_JSON replace kar:

{
    "1":"Transaction Request Date And Time",
    "2":"Transaction Success Date And Time",
    "3":"Merchant Order No",
    "4":"SBIEPAY ORDER ID",
    "5":"Cust Id",
    "6":"ATRN",
    "7":"Gateway Trace Number",
    "8":"Pay Mode Code",
    "9":"Gateway Name",
    "10":"Pay Proc",
    "11":"Transaction Currency",
    "12":"Merchant Order Amount",
    "13":"Gateway Posting Amount",
    "14":"Commission",
    "15":"GST",
    "16":"Order Status",
    "17":"Transaction Status",
    "18":"Settlement Status",
    "19":"Refund Status",
    "20":"Chargeback Status",
    "21":"Amount Refunded",
    "22":"Amount Chargeback",
    "23":"CIN Number",
    "24":"Merchant Other Details",
    "25":"Settlement Date"
}

He sagle changes reviewer chya comment nusar ahet.




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
