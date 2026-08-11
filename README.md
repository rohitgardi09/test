private String parseReportFileName(GstReportStatusDto gstReportStatusDto, String gstProcessingStatus) {

    String fileName = gstReportStatusDto.getS3Path();

    log.info("Parsing GST report file. status={}, fileName={}", gstProcessingStatus, fileName);

    if (ReportStatus.SUCCESS.getName().equals(gstProcessingStatus) && fileName.startsWith(SUCCESS_GST_REPORT_ACK_FILE_PREFIX)) {
        String parsedFileName = fileName.substring(SUCCESS_RES_TAX_LENGTH);
        log.info("GST success report file parsed successfully. parsedFileName={}", parsedFileName);
        return parsedFileName;
    } else if (ReportStatus.FAIL.getName().equals(gstProcessingStatus) && fileName.startsWith(ERROR_GST_REPORT_ACK_FILE_PREFIX)) {
        String parsedFileName = fileName.substring(FAIL_ERR_LENGTH);
        log.info("GST error report file parsed successfully. parsedFileName={}", parsedFileName);
        return parsedFileName;
    }

    log.error("Invalid GST report file or processing status. status={}, fileName={}", gstProcessingStatus, fileName);

    throw new ReportingException(NOT_FOUND_ERROR_CODE, MessageFormat.format(NOT_FOUND_ERROR_MESSAGE, gstProcessingStatus));
}

private void saveGstnReportInfo(GstReportStatusDto gstReportStatusDto, List<Object[]> gstnProcessingDtoList, String gstProcessingStatus) {

    String fileName = parseReportFileName(gstReportStatusDto,gstProcessingStatus);

    log.info("Fetching existing GST report. fileName={}", fileName);

    GstReportInfo existingReport = gstReportManagementDao.findReportTypeAndMonthYearByName(fileName).orElseThrow(() -> {
        log.error("GST Report not found. fileName={}", fileName);
        return new ReportingException(NOT_FOUND_ERROR_CODE, MessageFormat.format(NOT_FOUND_ERROR_MESSAGE, "GST Report"));
    });

    String s3Path = gstReportStatusDto.getS3Path();

    GstReportInfo gstReportInfoData = buildGstReportInfo(gstReportStatusDto, gstProcessingStatus, s3Path, existingReport);

    gstReportInfoRepository.save(gstReportInfoData);

    log.info("GST report info saved successfully. s3Path={}, status={}", s3Path, gstProcessingStatus);
}

private static GstReportInfo buildGstReportInfo(GstReportStatusDto gstReportStatusDto, String gstProcessingStatus, String s3Path, GstReportInfo existingReport) {
    return GstReportInfo.builder()
            .name(s3Path)
            .s3Path(s3Path)
            .reportType(existingReport.getReportType())
            .monthYear(existingReport.getMonthYear())
            .remark(gstReportStatusDto.getRemark())
            .recordType(RecordType.GST_RESPONSE_FILE)
            .totalCount(gstReportStatusDto.getTotalCount())
            .failedCount(gstReportStatusDto.getFailedCount())
            .inprogressCount(gstReportStatusDto.getInprogressCount())
            .successCount(gstReportStatusDto.getSuccessCount())
            .sftpPath(gstReportStatusDto.getSftpPath())
            .status(gstProcessingStatus)
            .build();
}
