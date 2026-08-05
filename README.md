
private RecordType validateRecordType(UUID gstInfoId) {

    GstReportInfoDto reportInfo = gstReportManagementDao.findById(gstInfoId);

    if (reportInfo == null) {
        throw new ReportingException("GST Report Info not found");
    }

    RecordType recordType = reportInfo.getRecordType();

    if (recordType == null) {
        throw new ReportingException("Record Type not found");
    }

    switch (recordType) {
        case Processing_File:
            log.info("Record Type : Processing_File");
            break;

        case Manual_File:
            log.info("Record Type : Manual_File");
            break;

        case Response_File:
            log.info("Record Type : Response_File");
            break;

        default:
            throw new ReportingException("Invalid Record Type : " + recordType);
    }

    return recordType;
}




कॉपी करण्यासाठी एकाच ठिकाणी देतो.

@RestController
@RequestMapping("/test/gst")
@RequiredArgsConstructor
public class GstTestController {

    private final GstReportInfoService gstReportInfoService;

    @PostMapping("/update-status")
    public ResponseEntity<String> updateStatus(@RequestBody GstReportStatusDto request) {
        gstReportInfoService.updateGstReportStatus(request);
        return ResponseEntity.ok("GST Report Status Updated Successfully");
    }
}

URL

POST http://localhost:8080/test/gst/update-status

Headers

Content-Type: application/json

SUCCESS Request Body

{
  "gstInfoId": "550e8400-e29b-41d4-a716-446655440000",
  "gstReportType": "SUCCESS_GST_REPORT",
  "s3Path": "gst-reports/2026/08/report.csv",
  "status": "SUCCESS",
  "remark": "Test API"
}

FAIL Request Body

{
  "gstInfoId": "550e8400-e29b-41d4-a716-446655440000",
  "gstReportType": "ERROR_GST_REPORT",
  "s3Path": "gst-reports/2026/08/error.csv",
  "status": "FAIL",
  "remark": "Test Failure"
}

टीप: हा API चालण्यासाठी gstInfoId हा GST_REPORT_INFO table मधला valid UUID असावा आणि s3Path मधील file उपलब्ध असावी. नाहीतर updateGstReportStatus() मध्ये exception येऊ शकतो.
