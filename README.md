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

