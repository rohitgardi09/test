private void createResponseFileReportInfo(GstReportStatusDto gstReportStatusDto) {

    String fileName = gstReportStatusDto.getS3Path();

    if (fileName != null && fileName.startsWith("RES_TAX_")) {
        fileName = fileName.substring(8);
    }

    GstReportInfo existingReport =
            gstReportManagementDao.findReportTypeAndMonthYearByName(fileName)
                    .orElseThrow(() -> new ReportingException(
                            NOT_FOUND_ERROR_CODE,
                            MessageFormat.format(NOT_FOUND_ERROR_MESSAGE, "GST Report")
                    ));

    GstReportInfo responseReport = new GstReportInfo();

    responseReport.setReportType(existingReport.getReportType());
    responseReport.setMonthYear(existingReport.getMonthYear());
    responseReport.setName(fileName);
    responseReport.setS3Path(fileName);
    responseReport.setRecordType(RecordType.GST_RESPONSE_FILE);

    gstReportInfoRepository.save(responseReport);
}






####
{
  "gstInfoId": 12345,
  "gstReportType": "SUCCESS_GST_REPORT",
  "s3Path": "RES_TAX_GST_REPORT_202608.csv"
}





@RestController
@RequestMapping("/test/gst-report")
@RequiredArgsConstructor
public class GstReportTestController {

    private final GstReportInfoService gstReportInfoService;

    @PostMapping("/status")
    public ResponseEntity<String> updateGstReportStatus(
            @RequestBody GstReportStatusDto gstReportStatusDto) {

        gstReportInfoService.updateGstReportStatus(gstReportStatusDto);

        return ResponseEntity.ok("GST report status processed successfully");
    }
}










public Optional<GstReportInfo> findReportTypeAndMonthYearByName(String fileName) {
    return gstReportInfoRepository.findReportTypeAndMonthYearByName(fileName);
}

####

@Query("""
       SELECT g
       FROM GstReportInfo g
       WHERE g.name = :name
       """)
Optional<GstReportInfo> findReportTypeAndMonthYearByName(@Param("name") String name);


####


private void updateGstReportStatus(GstReportStatusDto gstReportStatusDto) {

    switch (gstReportStatusDto.getGstReportType()) {

        case MERCHANT_GST_REPORT:
            log.info("Report Type : MERCHANT_GST_REPORT");
            gstReportManagementDao.updateGstReportStatus(gstReportStatusDto);
            break;

        case CUSTOMER_GST_REPORT:
            log.info("Report Type : CUSTOMER_GST_REPORT");
            gstReportManagementDao.updateGstReportStatus(gstReportStatusDto);
            break;

        case SUCCESS_GST_REPORT:
            log.info("Report Type : SUCCESS_GST_REPORT");
            updateReportDetailsFromFileName(gstReportStatusDto);
            updateGstReportDetail(gstReportStatusDto, ReportStatus.SUCCESS.getName());
            break;

        case ERROR_GST_REPORT:
            log.info("Report Type : ERROR_GST_REPORT");
            updateReportDetailsFromFileName(gstReportStatusDto);
            updateGstReportDetail(gstReportStatusDto, ReportStatus.FAIL.getName());
            break;

        default:
            throw new ReportingException(
                    NOT_FOUND_ERROR_CODE,
                    MessageFormat.format(NOT_FOUND_ERROR_MESSAGE,
                            gstReportStatusDto.getGstReportType()));
    }
}





#######

private void updateReportDetailsFromFileName(GstReportStatusDto gstReportStatusDto) {

    String fileName = gstReportStatusDto.getS3Path();

    if (fileName != null && fileName.startsWith("RES_TAX_")) {
        fileName = fileName.substring(8);
    }

    GstReportInfo gstReportInfo = gstReportManagementDao
            .findReportTypeAndMonthYearByName(fileName)
            .orElseThrow(() -> new ReportingException(
                    NOT_FOUND_ERROR_CODE,
                    MessageFormat.format(NOT_FOUND_ERROR_MESSAGE, "GST Report")));

    gstReportStatusDto.setGstReportType(gstReportInfo.getReportType());
    gstReportStatusDto.setMonthYear(gstReportInfo.getMonthYear());
}










@Repository
public interface GstReportInfoRepository extends JpaRepository<GstReportInfo, UUID> {

    @Query("""
            SELECT g
            FROM GstReportInfo g
            WHERE g.name = :name
            """)
    Optional<GstReportInfo> findReportTypeAndMonthYearByName(@Param("name") String name);

}

dao



public Optional<GstReportInfo> findReportTypeAndMonthYearByName(String fileName) {

    log.info("Fetching Report Type and Month Year for file : {}", fileName);

    return gstReportInfoRepository.findReportTypeAndMonthYearByName(fileName);
}






public void updateGstReportStatus(GstReportStatusDto gstReportStatusDto) {

    log.info("Updating GST Report Status : {}", gstReportStatusDto.getGstInfoId());

    String fileName = gstReportStatusDto.getS3Path();

    // RES_TAX_ remove
    if (fileName != null && fileName.startsWith("RES_TAX_")) {
        fileName = fileName.substring("RES_TAX_".length());
    }

    GstReportInfo gstReportInfo = gstReportManagementDao
            .findReportTypeAndMonthYearByName(fileName)
            .orElseThrow(() -> new ReportingException(
                    NOT_FOUND_ERROR_CODE,
                    MessageFormat.format(NOT_FOUND_ERROR_MESSAGE, "GST Report")));

    GstReportType reportType = gstReportInfo.getReportType();
    String monthYear = gstReportInfo.getMonthYear();

    gstReportStatusDto.setGstReportType(reportType);
    gstReportStatusDto.setMonthYear(monthYear);

    switch (reportType) {

        case MERCHANT_GST_REPORT:
            log.info("Merchant GST Report");
            gstReportManagementDao.updateGstReportStatus(gstReportStatusDto);
            break;

        case CUSTOMER_GST_REPORT:
            log.info("Customer GST Report");
            gstReportManagementDao.updateGstReportStatus(gstReportStatusDto);
            break;

        case SUCCESS_GST_REPORT:
            updateGstReportDetail(gstReportStatusDto,
                    ReportStatus.SUCCESS.getName());
            break;

        case ERROR_GST_REPORT:
            updateGstReportDetail(gstReportStatusDto,
                    ReportStatus.FAIL.getName());
            break;

        default:
            throw new ReportingException(
                    NOT_FOUND_ERROR_CODE,
                    "Invalid Report Type");
    }
}

