GstReportInfo responseReport = new GstReportInfo();

responseReport.setName(gstReportStatusDto.getFileName());
responseReport.setS3Path(gstReportStatusDto.getS3Path());

responseReport.setReportType(gstReportStatusDto.getGstReportType());
responseReport.setMonthYear(gstReportStatusDto.getMonthYear());

responseReport.setRecordType(RecordType.GST_RESPONSE_FILE);

responseReport.setStatus(gstReportStatusDto.getStatus());
responseReport.setRemark(gstReportStatusDto.getRemark());

// जर DTO मध्ये हे values available असतील तर
responseReport.setTotalCount(gstReportStatusDto.getTotalCount());
responseReport.setFailedCount(gstReportStatusDto.getFailedCount());
responseReport.setInProgressCount(gstReportStatusDto.getInProgressCount());
responseReport.setSuccessCount(gstReportStatusDto.getSuccessCount());

gstReportInfoRepository.save(responseReport);
