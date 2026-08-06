
public void updateGstReportStatus(GstReportStatusDto gstReportStatusDto) {

    log.info("GstReportInfoService - GstReportStatus updating for id : {}",
            gstReportStatusDto.getGstInfoId());

    // Response/S3 मधून आलेला file name
    String fileName = gstReportStatusDto.getS3Path();

    // RES_TAX_ prefix remove कर
    if (fileName != null && fileName.startsWith("RES_TAX_")) {
        fileName = fileName.substring("RES_TAX_".length());
    }

    // DB मधून report info शोध
    GstReportInfo gstReportInfo = gstReportManagementDao
            .findByName(fileName)
            .orElseThrow(() ->
                    new ReportingException(
                            NOT_FOUND_ERROR_CODE,
                            MessageFormat.format(NOT_FOUND_ERROR_MESSAGE, "GST report")));

    // DB मधून आलेला report type DTO मध्ये set कर
    gstReportStatusDto.setGstReportType(gstReportInfo.getReportType());

    switch (gstReportInfo.getReportType()) {

        case CUSTOMER_GST_REPORT:
            gstReportManagementDao.updateGstReportStatus(gstReportStatusDto);
            break;

        case MERCHANT_GST_REPORT:
            gstReportManagementDao.updateGstReportStatus(gstReportStatusDto);
            break;

        case SUCCESS_GST_REPORT:
            updateGstReportDetail(gstReportStatusDto, ReportStatus.SUCCESS.getName());
            break;

        case ERROR_GST_REPORT:
            updateGstReportDetail(gstReportStatusDto, ReportStatus.FAIL.getName());
            break;

        default:
            throw new ReportingException(
                    NOT_FOUND_ERROR_CODE,
                    "Invalid Report Type");
    }
}



############








public void updateGstReportStatus(GstReportStatusDto gstReportStatusDto) {

    log.info("GstReportInfoService - GstReportStatus updating for id : {}",
            gstReportStatusDto.getGstInfoId());

    // Response मधून आलेला file name
    String fileName = gstReportStatusDto.getFileName();

    // RES_TAX_ prefix remove कर
    if (fileName != null && fileName.startsWith("RES_TAX_")) {
        fileName = fileName.substring("RES_TAX_".length());
    }

    // DB मधून Report Type आणि Month Year आण
    GstReportInfo gstReportInfo =
            gstReportManagementDao.findByName(fileName);

    gstReportStatusDto.setGstReportType(gstReportInfo.getReportType());
    gstReportStatusDto.setMonthYear(gstReportInfo.getMonthYear());

    switch (gstReportStatusDto.getGstReportType()) {

        case CUSTOMER_GST_REPORT:
            gstReportManagementDao.updateGstReportStatus(gstReportStatusDto);
            break;

        case MERCHANT_GST_REPORT:
            gstReportManagementDao.updateGstReportStatus(gstReportStatusDto);
            break;

        case SUCCESS_GST_REPORT:
            updateGstReportDetail(gstReportStatusDto, ReportStatus.SUCCESS.getName());
            break;

        case ERROR_GST_REPORT:
            updateGstReportDetail(gstReportStatusDto, ReportStatus.FAIL.getName());
            break;

        default:
            throw new IllegalArgumentException(
                    "Invalid Report Type : " + gstReportStatusDto.getGstReportType());
    }
}
*#######

GstReportInfo findByName(String fileName);


####

SELECT REPORT_TYPE,
       MONTH_YEAR
FROM GST_REPORT_INFO
WHERE NAME = :fileName;






UPDATE GST_REPORT_INFO
SET RECORD_TYPE = 'Customer'
WHERE RECORD_TYPE IS NULL;

COMMIT;



ALTER TABLE GST_REPORT_INFO
MODIFY (RECORD_TYPE VARCHAR2(200) NOT NULL);





private void processReportStatus(GstReportStatusDto gstReportStatusDto) {

    RecordType recordType = validateRecordType(gstReportStatusDto.getGstInfoId());

    switch (recordType) {

        case Processing_File:
            log.info("Record Type : Processing_File");
            gstReportManagementDao.updateGstReportStatus(gstReportStatusDto);
            break;

        case Manual_File:
            log.info("Record Type : Manual_File");
            gstReportManagementDao.updateGstReportStatus(gstReportStatusDto);
            break;

        case Response_File:
            log.info("Record Type : Response_File");
            gstReportManagementDao.updateGstReportStatus(gstReportStatusDto);
            break;

        default:
            throw new ReportingException(
                    ReportingErrorCode.INVALID_REQUEST_ERROR_CODE,
                    "Invalid Record Type");
    }
}
