private void updateReportDetailsFromFileName(
        GstReportStatusDto gstReportStatusDto,
        List<Object[]> gstnProcessingDtoList,
        String gstProcessingStatus) {

    try {
        String fileName = gstReportStatusDto.getS3Path();

        if (fileName != null && fileName.startsWith("RES_TAX_")) {
            fileName = fileName.substring(8);
        }

        GstReportInfo existingReport =
                gstReportManagementDao.findReportTypeAndMonthYearByName(fileName);

        if (existingReport == null) {
            log.error("GST Report not found for file name: {}", fileName);
            return;
        }

        String s3Path = gstReportStatusDto.getS3Path();

        GstReportInfo responseReport = GstReportInfo.builder()
                .name(s3Path)
                .s3Path(s3Path)

                // Existing table मधून
                .reportType(existingReport.getReportType())
                .monthYear(existingReport.getMonthYear())

                // Response details
                .status(gstProcessingStatus)
                .remark(gstReportStatusDto.getRemark())

                // Counts
                .totalCount(gstReportStatusDto.getTotalCount())
                .failedCount(gstReportStatusDto.getFailedCount())
                .inprogressCount(gstReportStatusDto.getInprogressCount())
                .successCount(gstReportStatusDto.getSuccessCount())

                .sftpPath(gstReportStatusDto.getSftpPath())
                .recordType(RecordType.GST_RESPONSE_FILE)
                .build();

        gstReportInfoRepository.save(responseReport);

    } catch (Exception ex) {
        log.error(
                "Error while creating GST response report for S3 path: {}",
                gstReportStatusDto.getS3Path(),
                ex
        );
        throw ex;
    }
}






#####
GstReportInfo responseReport = new GstReportInfo();

String s3Path = gstReportStatusDto.getS3Path();

responseReport.setName(s3Path);
responseReport.setS3Path(s3Path);

// Existing GST report table मधून values घ्या
responseReport.setReportType(existingReport.getReportType());
responseReport.setMonthYear(existingReport.getMonthYear());

responseReport.setRecordType(RecordType.GST_RESPONSE_FILE);

responseReport.setStatus(gstReportStatusDto.getStatus());
responseReport.setRemark(gstReportStatusDto.getRemark());

responseReport.setTotalCount(gstReportStatusDto.getTotalCount());
responseReport.setFailedCount(gstReportStatusDto.getFailedCount());
responseReport.setInprogressCount(gstReportStatusDto.getInprogressCount());
responseReport.setSuccessCount(gstReportStatusDto.getSuccessCount());

gstReportInfoRepository.save(responseReport);

#₹₹₹₹






GstReportInfo responseReport = new GstReportInfo();

String s3Path = gstReportStatusDto.getS3Path();

responseReport.setName(s3Path);
responseReport.setS3Path(s3Path);

responseReport.setReportType(gstReportStatusDto.getGstReportType());
responseReport.setMonthYear(gstReportStatusDto.getMonthYear());

responseReport.setRecordType(RecordType.GST_RESPONSE_FILE);

responseReport.setStatus(gstReportStatusDto.getStatus());
responseReport.setRemark(gstReportStatusDto.getRemark());

responseReport.setTotalCount(gstReportStatusDto.getTotalCount());
responseReport.setFailedCount(gstReportStatusDto.getFailedCount());
responseReport.setInprogressCount(gstReportStatusDto.getInprogressCount());
responseReport.setSuccessCount(gstReportStatusDto.getSuccessCount());

gstReportInfoRepository.save(responseReport);
