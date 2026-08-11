private String parseReportFileName(GstReportStatusDto gstReportStatusDto, String gstProcessingStatus) {

    String fileName = gstReportStatusDto.getS3Path();

    log.info("Parsing GST report file name for status: {}", gstProcessingStatus);

    if (ReportStatus.SUCCESS.getName().equals(gstProcessingStatus)
            && fileName.startsWith(SUCCESS_GST_REPORT_ACK_FILE_PREFIX)) {
        String parsedFileName = fileName.substring(SUCCESS_RES_TAX_LENGTH);
        log.info("GST success report file name parsed successfully: {}", parsedFileName);
        return parsedFileName;

    } else if (ReportStatus.FAIL.getName().equals(gstProcessingStatus)
            && fileName.startsWith(ERROR_GST_REPORT_ACK_FILE_PREFIX)) {
        String parsedFileName = fileName.substring(FAIL_ERR_LENGTH);
        log.info("GST error report file name parsed successfully: {}", parsedFileName);
        return parsedFileName;
    }

    log.error("Invalid GST report file name or processing status. File: {}, Status: {}",
            fileName, gstProcessingStatus);

    throw new ReportingException(
            NOT_FOUND_ERROR_CODE,
            MessageFormat.format(NOT_FOUND_ERROR_MESSAGE, gstProcessingStatus));
}

private void saveGstnReportInfo(GstReportStatusDto gstReportStatusDto,
                                List<Object[]> gstnProcessingDtoList,
                                String gstProcessingStatus) {

    String fileName = parseReportFileName(gstReportStatusDto, gstProcessingStatus);

    log.info("Fetching existing GST report info for file: {}", fileName);

    GstReportInfo existingReport = gstReportManagementDao
            .findReportTypeAndMonthYearByName(fileName)
            .orElseThrow(() -> {
                log.error("GST report not found for file: {}", fileName);
                return new ReportingException(
                        NOT_FOUND_ERROR_CODE,
                        MessageFormat.format(NOT_FOUND_ERROR_MESSAGE, "GST Report"));
            });

    String s3Path = gstReportStatusDto.getS3Path();

    GstReportInfo gstReportInfoData =
            buildGstReportInfo(gstReportStatusDto, gstProcessingStatus, s3Path, existingReport);

    gstReportInfoRepository.save(gstReportInfoData);

    log.info("GST report info saved successfully for file: {}, status: {}",
            s3Path, gstProcessingStatus);
}
