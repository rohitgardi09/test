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
