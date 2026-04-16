Hi Team,

The latest release V1.5.0 is now ready for testing.
Features have been synced from develop to release branch.

Please find below the list of changes included in this release:

- Technical Upgrade: Spring Boot 3.5.8, Vulnerability Fixes, JSON Property fix, and Performance Optimization
  https://gitlab.epay.sbi/epay/microservices/epay_reports_service/-/issues/178

- Add UTRN Parameter in Settlement API dynamically (v1)
  https://gitlab.epay.sbi/epay/microservices/epay_reports_service/-/merge_requests/522

- Update Kafka dev certificate to enable local connectivity
  https://gitlab.epay.sbi/epay/microservices/epay_reports_service/-/issues/191

- Refactor report status handling to differentiate No_record_found and GENERATION_FAILED
  https://gitlab.epay.sbi/epay/microservices/epay_reports_service/-/issues/325

- Fix sensitive information exposure (JWT Token in DFRA report)
  https://gitlab.epay.sbi/epay/microservices/epay_reports_service/-/issues/312

- Add settlement date in Merchant Panel (Transaction → Recent page)
  https://gitlab.epay.sbi/epay/microservices/epay_reports_service/-/issues/278

- Add observability logging for Kafka message publish completion
  https://gitlab.epay.sbi/epay/microservices/epay_reports_service/-/issues/327

- Fix empty CSV issue in GST zip download (multiple months selection)
  https://gitlab.epay.sbi/epay/microservices/epay_reports_service/-/issues/323

- Ensure transaction details are displayed with two decimal precision
  https://gitlab.epay.sbi/epay/microservices/epay_reports_service/-/issues/303

- Update controller: invoice and GST download methods (ResponseEntity to void)
  https://gitlab.epay.sbi/epay/microservices/epay_reports_service/-/issues/176

- Fix missing synonyms for APPADMIN and APPREAD
  https://gitlab.epay.sbi/epay/microservices/epay_reports_service/-/issues/272

- Add settlement date in transaction report (/report/management)
  https://gitlab.epay.sbi/epay/microservices/epay_reports_service/-/issues/280

- New report added: daily_summary_refund
  https://gitlab.epay.sbi/epay/microservices/epay_reports_service/-/work_items/197

- Fix negative pending refund amount issue (Production)
  https://gitlab.epay.sbi/epay/microservices/epay_merchant_service/-/issues/599

Kindly start testing from your end and report any issues observed.

Thanks & Regards,
Rohit
