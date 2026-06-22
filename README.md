@GetMapping("/download")
public void downloadFile(HttpServletResponse response) throws IOException {

    String filePath = "C:/temp/sample.pdf";
    File file = new File(filePath);

    response.setContentType("application/octet-stream");
    response.setHeader(
            "Content-Disposition",
            "attachment; filename=\"" + file.getName() + "\"");

    try (InputStream inputStream = new FileInputStream(file);
         OutputStream outputStream = response.getOutputStream()) {

        byte[] buffer = new byte[8192];
        int bytesRead;

        while ((bytesRead = inputStream.read(buffer)) != -1) {
            outputStream.write(buffer, 0, bytesRead);
        }

        outputStream.flush();
    }
}




............
package com.epay.merchant.controller;

import com.epay.merchant.service.InvoiceService;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * Class Name: InvoiceController
 * *
 * Description:This class handles all incoming requests related to invoice management and provides appropriate responses.
 * It interacts with the InvoiceService to process and retrieve invoice data as required by the client
 * *
 * Author: Lohith M P
 * <p>
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * *
 * Version:1.0
 */
@RequiredArgsConstructor
@RestController
@RequestMapping("/invoice")
public class InvoiceController {
    private final InvoiceService invoiceService;

    /**
     * Generates the merchant gst invoice based on the provided merchant ID (MID) and report dates.
     * In case of error: A response containing a success or error message related to the invoice generation.
     *
     * @param mId          The merchant ID (MID) for which the invoice is to be generated.
     * @param reportMonths The request body containing the details needed to identify and download the report, such as file path or report type.
     * @param response     The HTTP response object used to send the generated invoice file.
     */
    @PostMapping("/gst/{mId}")
    public void downloadGstInvoice(HttpServletResponse response, @PathVariable String mId, @RequestBody List<String> reportMonths) {
        invoiceService.downloadMerchantGstInvoice(response, mId, reportMonths);
    }
}

==========================================================================================================================================================

package com.epay.merchant.service;

import com.epay.merchant.dao.AlertDao;
import com.epay.merchant.dao.InvoiceDao;
import com.epay.merchant.dto.AlertMasterDto;
import com.epay.merchant.dto.GstInvoiceDto;
import com.epay.merchant.validator.InvoiceValidator;
import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.transaction.Transactional;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.util.List;

import static com.epay.merchant.util.MerchantConstant.GST_INVOICE;

@Service
@RequiredArgsConstructor
public class InvoiceService {
    private final LoggerUtility log = LoggerFactoryUtility.getLogger(this.getClass());
    private final InvoiceDao invoiceDao;
    private final InvoiceValidator invoiceValidator;
    private final FileGenerationService fileGenerationService;
    private final AlertDao alertDao;

    /**
     * Validate the requested gst invoice is available to download , if present then set it in the http servlet response.
     *
     * @param response     HttpServletResponse
     * @param mId          String
     * @param reportMonths report months
     */
    public void downloadMerchantGstInvoice(HttpServletResponse response, String mId, List<String> reportMonths) {
        log.info("Received Gst invoice download for mId {}, reportMonths {}", mId, reportMonths);
        invoiceValidator.validateReportMonths(reportMonths);
        invoiceValidator.validateMid(mId);
        List<String> s3FilePaths = invoiceDao.getS3Path(mId, reportMonths);
        fileGenerationService.downloadGstInvoiceFiles(response, s3FilePaths);
    }

    /**
     * saveGstInvoiceDownload requested to store GST invoice details and GST invoice alert format.
     *
     * @param gstInvoiceDto     GstInvoiceDto
     */
    @Transactional
    public void saveGstInvoiceDownload(GstInvoiceDto gstInvoiceDto) {
        log.info("Received Gst invoice download event for generate notification, mId {}, reportMonths {}", gstInvoiceDto.getMId(), gstInvoiceDto.getMonthYear());
        invoiceDao.saveGstInvoiceDownloadDetails(gstInvoiceDto);
        AlertMasterDto alertMasterDto = alertDao.getAlertMaster(GST_INVOICE);
        alertDao.saveAlertManagement(gstInvoiceDto.getMId(), alertMasterDto);
    }
}



==========================================================================================================================================================

package com.epay.merchant.validator;

import com.epay.merchant.dao.ValidationDao;
import com.epay.merchant.exception.ValidationException;
import com.epay.merchant.util.EPayIdentityUtil;
import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;

import java.time.YearMonth;
import java.time.format.DateTimeParseException;
import java.util.ArrayList;
import java.util.List;

import static com.epay.merchant.util.DateTimeUtils.FORMATTER_YYYY_MM;
import static com.epay.merchant.util.ErrorConstants.*;
import static com.epay.merchant.util.MerchantConstant.*;


/**
 * Class Name: ReportManagementValidator
 * *
 * Description: This class is responsible for validating all report-related download requests, ensuring that the provided
 * report details, such as report type, format, date range, and merchant ID (MId), are valid. It also validates if the
 * requested reports exist and are accessible for download.
 * *
 * Author: Lohith M P
 * <p>
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * *
 * Version:1.0
 */
@Component
@RequiredArgsConstructor
public class InvoiceValidator extends BaseValidator {

    private final LoggerUtility logger = LoggerFactoryUtility.getLogger(this.getClass());
    private final ValidationDao validationDao;

    /**
     * Validates a download request by checking the mandatory fields and ensuring the report exists in the system.
     *
     * @param yearMonths List of invoice year months.
     * @throws ValidationException if any validation fails.
     */
    public void validateReportMonths(List<String> yearMonths) {
        logger.info("Initiated Merchant gst invoice request validation");
        errorDtoList = new ArrayList<>();

        validationMandatory(yearMonths);
        YearMonth currentMonth = YearMonth.now();

        yearMonths.forEach(requestedYearMonth -> {
            try {
                checkForLeadingTrailingAndSingleSpace(requestedYearMonth, DATE);
                throwIfErrors();

                YearMonth yearMonth = YearMonth.parse(requestedYearMonth, FORMATTER_YYYY_MM);
                int year = yearMonth.getYear();

                if (year < INVALID_YEAR || year > currentMonth.getYear()) {
                    //validate year range (adjust lower bound as needed)
                    addError(INVALID_ERROR_CODE, INVALID_ERROR_MESSAGE, MONTH_YEAR, INVALID_YEAR_MESSAGE);
                } else if (year == currentMonth.getYear() && yearMonth.getMonthValue() > currentMonth.getMonthValue()) {
                    //validate month range
                    addError(INVALID_ERROR_CODE, INVALID_ERROR_MESSAGE, MONTH_YEAR, INVALID_MONTH_MESSAGE);
                }

            } catch (DateTimeParseException e) {
                addError(DATE_FORMAT_ERROR_CODE, DATE_FORMAT_ERROR_MESSAGE, DATE_FORMAT);
            }

        });
        throwIfErrors();
    }

    /**
     * Validates all mandatory fields
     *
     * @param dates List<String>
     */
    private void validationMandatory(List<String> dates) {
        checkMandatoryCollection(dates, MONTH_YEAR);
        dates.forEach(date -> {
            checkMandatoryField(date, MONTH_YEAR);
            throwIfErrors();
        });
    }

    /**
     * validates the MId.
     *
     * @param mId The request containing mId.
     * @throws ValidationException if any validation fails.
     */
    public void validateMid(String mId) {
        logger.info("Validating mId.");
        validateMIdValue(mId);
        validationDao.validatedMId(EPayIdentityUtil.getUserPrincipal().getUsername(), mId);
    }


    /**
     * validates the MId value.
     *
     * @param mId The request containing mId.
     * @throws ValidationException if any validation fails.
     */
    private void validateMIdValue(String mId) {
        errorDtoList = new ArrayList<>();
        checkMandatoryField(mId, MID);
        throwIfErrors();
        checkForLeadingTrailingAndSingleSpace(mId, MID);
        throwIfErrors();
        validateFixedFieldLength(mId, MID_LENGTH, MID);
        throwIfErrors();
        validateFieldWithRegex(mId, MID_LENGTH, MID_REGEX, MID, INVALID_FORMAT);
        throwIfErrors();
    }

}



==========================================================================================================================================================
package com.epay.merchant.service;

import com.epay.merchant.exception.MerchantException;
import com.epay.merchant.service.file.FileService;
import com.epay.merchant.util.MerchantUtil;
import com.epay.merchant.util.file.generator.ZIPGenerator;
import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.apache.commons.io.FilenameUtils;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Service;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import static com.epay.merchant.util.ErrorConstants.GENERIC_ERROR_CODE;
import static com.epay.merchant.util.ErrorConstants.GENERIC_ERROR_MESSAGE;
import static com.epay.merchant.util.MerchantConstant.APPLICATION_ZIP;
import static com.epay.merchant.util.MerchantConstant.MERCHANT_GST_INVOICES_ZIP_NAME;

/**
 * Class Name: FileGeneratorService
 * Description: This service handles the logic for generating files, including the creation of individual files
 * and the generation of zip archives. It uses `FileGenerator` and `ZipFileGenerator` components to produce
 * file outputs in different formats and efficiently manages file generation processes.
 * Author: Lohith M P
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * Version: 1.0
 */
@RequiredArgsConstructor
@Service
public class FileGenerationService {

    private final LoggerUtility log = LoggerFactoryUtility.getLogger(this.getClass());
    private final FileService s3Service;

    /**
     * Building content type and setting file in response.
     *
     * @param response    HttpServletResponse
     * @param s3FilePaths String
     */
    public void downloadGstInvoiceFiles(HttpServletResponse response, List<String> s3FilePaths) {
        log.info("Started downloading Pdf Files for s3FilePaths: {}", s3FilePaths);
        try {
            if (s3FilePaths.size() == 1) {
                // if single file, download and set it to the response
                downloadFileAndWriteToResponse(response, s3FilePaths.getFirst());
            } else {
                // if multiple files, download and zip them, then set the zip to the response
                downloadZipAndWriteToResponse(response, s3FilePaths);
            }

        } catch (IOException e) {
            log.error("Failed to read file[{}] from S3 with error: {}", e.getMessage());
            throw new MerchantException(GENERIC_ERROR_CODE, GENERIC_ERROR_MESSAGE);
        }
    }

    private void downloadFileAndWriteToResponse(HttpServletResponse response, String s3Path) throws IOException {
        log.info("Downloading single file");
        try (InputStream inputStream = s3Service.readFile(s3Path)) {
            MerchantUtil.setHeader(response, MediaType.APPLICATION_PDF_VALUE, FilenameUtils.getName(s3Path));
            inputStream.transferTo(response.getOutputStream());
            response.getOutputStream().close();
        }
    }

    private void downloadZipAndWriteToResponse(HttpServletResponse response, List<String> s3Paths) throws IOException {
        Map<String, InputStream> s3FilesMap = new HashMap<>();
        log.info("Downloading multiple files");
        for (String path : s3Paths) {
            InputStream inputStream = s3Service.readFile(path);
            String fileName = FilenameUtils.getName(path);
            s3FilesMap.put(fileName, inputStream);
        }
        log.info("Zipping downloaded files");
        ByteArrayOutputStream zipBytes = ZIPGenerator.zipS3Files(s3FilesMap);
        MerchantUtil.setHeader(response, APPLICATION_ZIP, MERCHANT_GST_INVOICES_ZIP_NAME);
        zipBytes.writeTo(response.getOutputStream());
        response.flushBuffer();
    }
}




==========================================================================================================================================================

    /**
     * GetAlertMaster - get Alert Format for notification
     * @param alertName String
     * @return AlertMasterDto object.
     */
    public AlertMasterDto getAlertMaster(String alertName) {
        log.info("Fetch GST invoice alert format for notification.");
        AlertMaster alertMaster = alertMasterRepository.findByName(alertName).orElseThrow(()-> new MerchantException(ErrorConstants.NOT_FOUND_ERROR_CODE, MessageFormat.format(ErrorConstants.NOT_FOUND_ERROR_MESSAGE, "Alert name  "+  MerchantConstant.REPORT_GENERATION)));
        return alertMapper.mapAlertMasterEntityToList(alertMaster);
    }

    /**
     * saveAlertManagement - save Alert Format for notification
     * @param mId String   - The merchant ID associated with the alert.
     * @param alertMasterDto AlertMasterDto object.
     */
    public void saveAlertManagement(String mId, AlertMasterDto alertMasterDto) {
        log.info("Save GST invoice alert format for notification.");
        AlertManagement alertManagement = AlertManagement.builder().alertId(alertMasterDto.getId()).mId(mId).alertDescription(alertMasterDto.getDescription()).alertIdentifier(MerchantUtil.generateUnique12DigitNumber()).build();
        alertManagementRepository.save(alertManagement);
    }

==========================================================================================================================================================

