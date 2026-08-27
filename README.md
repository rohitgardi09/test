public static final String JDBC_MERCHANT_GST_REPORT = """
        SELECT
        BGL_ACCOUNT_NO as bglAccountNo,
        INCOME_BOOKING_BRANCH as incomeBookingBranch,
        TRANSACTING_BRANCH as transactingBranch,
        'EPY11777' || FUN_GET_TIME_NANOO() as uniqueReferenceNo,
        CUSTOMER_ACCOUNT_NUMBER as customerAccountNumber,
        CUSTOMER_GSTIN as customerGSTIN,
        NAME_OF_CUSTOMER as nameOfCustomer,
        MIN(DATE_OF_SUPPLY) as dateOfSupply,
        COMMODITY_CODE as commodityCode,
        HSN_OR_SAC as hsnOrSac,
        SUM(NVL(VALUE,0)) as value,
        SUM(NVL(TAXABLE_VALUE,0)) as taxableValue,
        SETTLED_DATE,
        SETTLEMENT_STATUS
        FROM CUSTOMER_GST_REPORT_VIEW
        WHERE SETTLED_DATE >=
        ((TO_DATE(:monthYear,'YYYYMM') - DATE '1970-01-01') * 86400 * 1000)
        AND SETTLED_DATE <
        ((ADD_MONTHS(TO_DATE(:monthYear,'YYYYMM'),1) - DATE '1970-01-01') * 86400 * 1000)
        GROUP BY BGL_ACCOUNT_NO, INCOME_BOOKING_BRANCH, TRANSACTING_BRANCH,
        CUSTOMER_ACCOUNT_NUMBER, CUSTOMER_GSTIN, NAME_OF_CUSTOMER,
        COMMODITY_CODE, HSN_OR_SAC, DATE_OF_SUPPLY
        """;
