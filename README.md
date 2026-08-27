/**
 * POST /gst/upload
 *
 * Description:
 * Uploads a GST report file manually for the specified report type and month-year.
 * The request accepts GST report details and the report file using multipart/form-data.
 *
 * Example:
 * POST /gst/upload
 * Content-Type: multipart/form-data
 *
 * gstUploadManualDto:
 * Description: GST report details including report type, month-year and remark.
 *
 * {
 *   "reportType": "MERCHANT_GST_REPORT",
 *   "monthYear": "202606",
 *   "remark": "Done"
 * }
 *
 * file:
 * Description: GST report file to be uploaded in CSV format.
 *
 * DET_CONS_GSTR1_19082026_EPY_19938.csv
 */
