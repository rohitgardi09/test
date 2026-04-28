
RELEASE NOTE
Release Summary
This release delivers critical bug fixes, system optimizations, and platform enhancements, along with dependency upgrades to improve security, performance, and maintainability.
Scope of Release
1. Functional Enhancements
Auditing implemented for Token Management using Hibernate
Merchant bank account onboarding flow enhanced
Resend OTP functionality introduced
User creation password template added
Kafka publish completion logging added
2. Bug Fixes
Resolved issue in forgot password flow when using email ID
Fixed missing Lombok constructor annotations across request classes
Addressed JSON property handling issues
Corrected missing synonym configurations
3. Performance & Refactoring
Optimized token validation query
Refactored captcha generation using configuration-based approach
Renamed column ENTITY_ID to USER_ID in Notification Management table
4. Technical Upgrades
Upgraded to Spring Boot 3.5.8 (includes vulnerability fixes)
Updated project dependencies and libraries
Improved overall application performance and stability
Database Changes
Column renamed:
ENTITY_ID → USER_ID (Notification Management table)
Liquibase scripts applied for schema updates
Impact Assessment
Improved authentication and token handling reliability
Enhanced onboarding and user management workflows
Better logging and monitoring capability
Reduced system vulnerabilities due to dependency upgrades
Risk & Mitigation
Risk: Dependency upgrade impact on existing modules
Mitigation: Regression testing completed across critical flows
Risk: DB schema change impact
Mitigation: Backward compatibility validated via Liquibase
Rollback Plan
Revert application to previous stable build
Rollback database changes using Liquibase rollback scripts
Disable new features (OTP resend, auditing) via configuration if required



समजलं 👍 — तू दिलेल्या points मधले technical details, config, logic, acceptance criteria अजून clearly add करून bank-level detailed release note बनवतो. हा version जास्त strong आहे 👇
📄 RELEASE NOTE – Report Service
🔖 Version: release/v1.5.0
📅 Release Date: (Add Date)
INTRODUCTION:
This release introduces critical improvements in the Report Service focusing on security compliance, performance optimization, API standardization, Kafka connectivity, and production issue resolution. It also includes enhancements to report status handling and transaction visibility, ensuring a better user experience and accurate system behavior.
Overview:
The release addresses multiple areas including dependency vulnerability fixes, JSON response consistency, Kafka SSL configuration for development, report status correction, and several production/UAT issues. Additionally, enhancements such as settlement date visibility and Kafka observability logging have been introduced to improve monitoring and reporting capabilities.
Code changes / Config changes / DB changes:
1. Spring Boot Upgrade, Vulnerability Fix & Performance Optimization
1.1 Upgraded Spring Boot from 3.5.6 to 3.5.8 to ensure compatibility with latest dependencies
1.2 Addressed vulnerabilities identified via Quay security scans
1.3 Upgraded logback-core to version 1.5.18 to mitigate known security issues
1.4 Fixed duplicate JSON field issue by removing mid and standardizing to mId across all APIs
1.5 Implemented Jackson Blackbird ObjectMapper for faster JSON serialization/deserialization
1.6 Improved API response time and reduced processing overhead
2. UTRN Parameter Enhancement
2.1 Implemented dynamic handling of UTRN parameter in settlement API
2.2 Ensures correct mapping and flexibility in settlement processing
2.3 Improves reconciliation and transaction traceability
3. Kafka Dev Certificate Update & Configuration Enhancement
Configuration Details:
YAML
kafka:
  bootstrapServers: dev-cluster-kafka-bootstrap-dev-kafka.apps.dev.sbiepay.sbi:443
  properties:
    security.protocol: SSL
    ssl:
      truststore:
        location: certs/kafka/dev-cluster-cluster-ca-cert.p12
        type: PKCS12
      keystore:
        location: certs/kafka/dev-cluster-clients-ca-cert.p12
        type: PKCS12
Changes Implemented:
3.1 Added new Kafka SSL certificate files under certs/kafka/
3.2 Updated application.yml to use relative certificate paths
3.3 Introduced utility method in ReportUtil.java:
Java
public static String getAbsolutePath(String fileName) {
    return new File(fileName).getAbsolutePath();
}
3.4 Updated KafkaProducerConfig:
Used dynamic absolute path for truststore and keystore
3.5 Enabled secure Kafka connection for local and dev environments
Acceptance Criteria:
Kafka producer and consumer APIs tested successfully
Verified message publishing and consumption
SSL connection established without errors
4. Report Status Refactoring
Problem:
Reports with no data were incorrectly marked as GENERATION_FAILED
Solution:
4.1 Introduced new status: NO_RECORD_FOUND
4.2 Updated backend logic:
Java
ReportStatus newStatus;
if (equalsIgnoreCase(NO_RECORD_FOUND, e.getErrorMessage())) {
    newStatus = ReportStatus.NO_RECORD_FOUND;
} else {
    newStatus = ReportStatus.GENERATION_FAILED;
}
4.3 Updated UI to display:
“No record found for the selection” instead of error
Acceptance Criteria:
Reports with no data show correct status
Actual failures still marked as GENERATION_FAILED
5. Security Fix – Sensitive Data Exposure
5.1 Removed JWT token exposure from DFRA report
5.2 Masked sensitive fields in API responses and logs
5.3 Ensured only authorized users can access sensitive data
5.4 Strengthened logging practices to avoid data leakage
6. Settlement Date Enhancement
Business Logic:
If transaction is settled → settlementDate populated
If failed/unsettled → settlementDate blank
Implementation:
6.1 Added settlementDate field in TransactionResponse DTO
6.2 Updated service layer to fetch and populate settlement date
6.3 Updated API contract (Swagger/OpenAPI)
6.4 UI updated to display settlement date
Acceptance Criteria:
Settlement date visible in transaction panel
Null values handled gracefully
7. Kafka Observability Logging
7.1 Implemented callback-based logging for Kafka publish events
7.2 Captures:
Message key
Topic name
Partition & offset
Processing duration
Error details (if failed)
7.3 Improves monitoring and debugging capability
8. GST ZIP Download Bug Fix
Issue:
ZIP files contained empty CSVs when multiple months selected
Fix:
8.1 Corrected batch processing logic
8.2 Ensured proper file writing before compression
8.3 Validated data presence in generated CSV files
9. Dashboard Decimal Fix
9.1 Ensured all monetary fields display 2 decimal precision:
Refund Adjusted
Net Settlement
Settled Amount
Pending Amount
10. Controller Update
10.1 Updated invoice and GST download APIs
10.2 Changed return type from ResponseEntity to void
10.3 Improved response handling and file streaming
11. DB Synonym Fix
11.1 Added missing synonyms:
APPADMIN
APPREAD
11.2 Ensured proper DB access and compatibility
12. Refund Pending Amount Fix
12.1 Fixed issue where pending refund amount was negative
12.2 Corrected calculation logic
12.3 Ensured proper display in reports
13. ISD Observations Fix
13.1 Fixed overly broad exception handling
13.2 Replaced generic catch blocks with specific exceptions
13.3 Improved logging for better traceability
13.4 Updated file handling logic (ZipFileGenerator)
Previous Impact:
Security vulnerabilities in dependencies
Duplicate and inconsistent JSON fields
Kafka SSL connectivity issues in dev
Incorrect report status for no data
Exposure of sensitive JWT data
GST ZIP files containing empty data
Incorrect decimal formatting
Negative refund pending values
Missing settlement date in transactions
Poor exception handling
Impact of Changes:
Improved system security and compliance
Enhanced API consistency
Faster response time and better performance
Accurate report status handling
Secure handling of sensitive data
Correct GST report generation
Improved financial data accuracy
Better transaction visibility
Stable Kafka integration
Enhanced logging and monitoring
Improved maintainability
Rollback Plan:
Revert Report Service to previous stable version
Restore old Kafka SSL configuration and certificates
Remove new fields (settlementDate, NO_RECORD_FOUND)
Revert JSON field naming changes
Rollback DB synonym updates
Restore previous exception handling logic
🔥 आता हा version bank / manager level detailed + audit ready आहे — काही कमी नाही.
जर अजून heavy करायचं असेल तर मी add करू शकतो:
Risk & Mitigation section
Deployment steps
Production validation checklist
फक्त सांग 👍
