


 ### 1.Spring Boot 3.5.8, Vulnerability Fixes, Fix JsonProperty issue and Performance Optimization
   This epic encompasses three critical technical upgrades to improve system security, consistency, and performance:

1.1. **Spring Boot Version Upgrade**
   * Update from Spring Boot `3.5.6` to `3.5.8`
   * Address Quay-reported dependency vulnerabilities
   * Specific focus on logback-core `1.5.18` security fixes
1.2. **JSON Field Standardization**
   * Resolve duplicate field naming in API responses
   * Standardize on `mId` field naming convention
   * Remove redundant `mid` fields
1.3. **Performance Enhancement**
   * Implement Jackson BlackBird ObjectMapper
   * Optimize JSON serialization/deserialization
   * Improve overall API response times
   
   
 ### 2. UTRN Parameter in settlement API dynamically v1


 ### 3.Update Kafka dev certificate to connect from local


### Update Certificates

---

For development, developer connects Dev env Kafka server which requires following SSL configuration:

> ```
>   kafka:
>     bootstrapServers: dev-cluster-kafka-bootstrap-dev-kafka.apps.dev.sbiepay.sbi:443
>     # Spring-Kafka SSL Configuration
>     properties:
>       security.protocol: SSL
>       ssl:
>         truststore:
>           location: C:/certs/kafka/dev-cluster-cluster-ca-cert.p12
>           password: ******
>           type: PKCS12
>         keystore:
>           location: C:/certs/kafka/dev-cluster-clients-ca-cert.p12
>           password: ******
>           type: PKCS12
> ```

#### Get new certs files and password from DevOps team and test it.

### Code improvement:

---

Update following logic for all developer fix:

3.1. Commit new cert file under certs\\kafka
3.2. Update application yaml file:
   1. truststore.location: \*\*certs/kafka/\*\*dev-cluster-cluster-ca-cert.p12
   2. keystore.location: \*\*certs/kafka/\*\*dev-cluster-clients-ca-cert.p12
3.3. Add following method in ReportUtil.java:

   > public static String getAbsolutePath(String fileName) {\
   > return new File(fileName).getAbsolutePath();\
   > }
3.4. Update KafkaProducerConfig.java

   > props.put(SslConfigs._SSL_TRUSTSTORE_LOCATION_CONFIG_, **_getAbsolutePath(_kafkaProducerSettings.getTrustLocation()_)_**);
   >
   > props.put(SslConfigs._SSL_KEYSTORE_LOCATION_CONFIG_, **_getAbsolutePath(_kafkaProducerSettings.getKeyLocation()_)_**);

#### Acceptance criteria:

* Test API which includes the KAFKA publisher and consumer.
* Attach screen-prints.
  
 ### 4.Refactor report status handling to differentiate 'No record found for the selection.' from 'GENERATION_FAILED'

**Description:**\
Currently, when a report returns no data, the `report-service` marks the status as `GENERATION_FAILED` based on the remark "No record found for the selection.", which is misleading for users.

**Goal:** Update the Java `report-service` logic to detect this specific remark and set a new status (`NO_RECORD_FOUND` or similar) instead of `GENERATION_FAILED`. The UI must be updated to display this status as a "No record found" message rather than an error.

**Tasks:**

4.1. **Java Service (Backend)**: Locate the report status management logic in `report-service`.
4.2. **Logic Change**: Implement a check in the result processor:

```Java
Add NO_RECORD_FOUND in ReportStatus.
Update ReportService.generateReport method's ReportingException catch block as below:
ReportStatus newStatus;
if (equalsIgnoreCase(NO_RECORD_FOUND, e.getErrorMessage())) {
    newStatus = ReportStatus.NO_RECORD_FOUND;
} else {
    newStatus = ReportStatus.GENERATION_FAILED;
}
reportManagementDao.updateStatusAndRemarks(reportManagementDto.getId(), newStatus, errorReason);
```

4.3. **UI Updates**: Update the frontend to handle the new `NO_RECORD_FOUND` status and display a friendly message instead of a red error icon.
4.4. **Testing**: Verify that reports with no data show "No record found" and actual failures still show "Generation Failed".

**Acceptance Criteria:**

* Reports with no data are not marked as `GENERATION_FAILED`.
* UI displays "No record found for the selection." when the new status is received.


 ### 5.UAT Issue: Sensitive information such as Jwt Token,is exposed in the DFRA report

- Do not show Jwt anywhere.
- Mask Jwt token. 
- Remove sensitive data from API responses and logs.
- Allow only authorized users to access such data.

 ### 6.Add settlement date in merchant panel transaction(transaction/recent/) page


### Business Requirement:

---

Settlement Date to be captured in transaction Report by following logic- If transaction is not settled, failed, settlement date will be blank. If transaction is settled, corresponding settlement date should be there.

---

**Description:**

This issue tracks the implementation of adding a `settlementDate` field to the JSON response of our `transaction/recent/` endpoint (and any related transaction retrieval endpoints) to enable the UI to display settlement information in the transaction panel.

**Goal:**\
Enhance transaction visibility by providing settlement details, improving reconciliation and user understanding.

**Technical Details (Backend - report-service):**

1. **Identify Data Source:** Determine where the `settlementDate` originates (e.g., another service, database column, calculated field).
2. **Update DTO/Model:** Add a `settlementDate` field (e.g., `LocalDate` or `ZonedDateTime`) to the `TransactionResponse` DTO/Model in the Java service.
3. **Fetch & Populate:** Modify the service logic for `transaction/recent/`to fetch and populate this date into the response object.
4. **API Contract Update:** Document the new field in the OpenAPI/Swagger spec for the `transaction/recent/` endpoint.
5. **Database (If needed):** Not required.

**Technical Details (Frontend - UI Panel):**

1. **API Integration:** Update the frontend service to consume the new `settlementDate` field from the `transaction/recent/` response.
2. **UI Component:** Add a new column or display element for `Settlement Date` in the existing transaction list/panel.
3. **Formatting:** Format the date as required (e.g., `YYYY-MM-DD`, `DD/MM/YYYY`).

**Acceptance Criteria (Backend):**

* `transaction/recent/` response includes `settlementDate` field when available.
* Swagger/OpenAPI spec updated.
* Unit/Integration tests pass. 

**Acceptance Criteria (Frontend):**

* `Settlement Date` column visible in the transaction panel.
* Correct date displayed for relevant transactions.
* Handles cases where `settlementDate` might be null/missing gracefully and display empty.

 ### 7.Add observability logging for Kafka message publish completion.
**Description**

Add callback-based logging to track Kafka message delivery completion, including processing duration, message key, topic, and offset metadata for successful deliveries, 
or detailed error information for failed attempts.

 ### 8.Bug: Zip download contains empty CSV files when selecting multiple GST months


**Description**\
When merchants select multiple GST months in the report generation panel, the resulting ZIP file downloads successfully but contains empty CSV files (0 KB), despite data existing for those periods individually. This appears to be a batch processing or file compression issue.

**Steps to Reproduce**

1. Navigate to the Merchant Panel \> GST Reports.
2. Select multiple months (e.g., Jan 2026, Feb 2026).
3. Click 'Download Zip'.
4. Extract the downloaded ZIP file.

**Expected Behavior**\
The CSV files inside the ZIP should contain sales/tax data for all selected months.

**Actual Behavior**\
The CSV files are generated but are empty.

 ### 9.Production Environment, Dashboard page>>Transaction Detail Must be Two Decimal.


IN Production Environment, Dashboard page Transaction Detail Must be Two Decimal.
TC Description: Transaction Details In The Dashboard Page Refund Adj, Net Settlement, Settled Amt Should be in Two Decimal Actual Behavior: 
| Refund Adjusted | Net Settlement Amt.(₹) | Settled Amt.(₹) | Pending Amt. (₹)

  ### 10.Updated controller invoice and gst download method return ResponseEntity to void


 ### 11.Report Service - Missing Synonym for APPADMIN and APPREAD


  ### 12.Pending Amount for refund is displayed with negative amount - Production Environment
  
 Description: Pending Amount  for refund is displayed with negative amount - Production
Steps to reproduce: download refund report.
Expected result: Amount should not displayed with negative amount
Actual Result: Amount is displayed with negative amount.

  ### 13.ISD Observations for Report Service
