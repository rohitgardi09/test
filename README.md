
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
