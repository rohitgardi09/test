// ============================================================
// GstReportCancelDto.java
// ============================================================

package com.epay.reporting.dto;

import lombok.Data;

@Data
public class GstReportCancelDto {

    private String remark;
}


// ============================================================
// GstReportController.java
// ============================================================

@PostMapping("/cancel/{id}")
@Operation(summary = "Cancel GST report")
public ReportingResponse<String> cancelGstReport(
        @PathVariable("id") String id,
        @Valid @RequestBody GstReportCancelDto request) {

    log.info("Received GST report cancellation request for id: {}", id);

    gstReportInfoService.cancelGstReport(id, request);

    return ReportingResponse.<String>builder()
            .status(ResponseStatus.SUCCESS)
            .message("GST report cancelled successfully")
            .build();
}


// ============================================================
// GstReportInfoService.java
// ============================================================

@Transactional
public void cancelGstReport(String id, GstReportCancelDto request) {

    log.info("Cancelling GST report for id: {}, remark: {}",
            id, request.getRemark());

    UUID reportId;

    try {
        reportId = UUID.fromString(id);
    } catch (IllegalArgumentException exception) {

        log.error("Invalid GST report id: {}", id, exception);

        throw new ReportingException(
                ErrorConstants.NOT_FOUND_ERROR_CODE,
                MessageFormat.format(
                        ErrorConstants.NOT_FOUND_ERROR_MESSAGE,
                        id));
    }

    GstReportInfo gstReportInfo = gstReportInfoDao.findById(reportId)
            .orElseThrow(() -> new ReportingException(
                    ErrorConstants.NOT_FOUND_ERROR_CODE,
                    MessageFormat.format(
                            ErrorConstants.NOT_FOUND_ERROR_MESSAGE,
                            id)));

    if (!ReportStatus.UPLOAD_SUCCESS.getName()
            .equals(gstReportInfo.getStatus())) {

        log.info(
                "GST report cannot be cancelled. id: {}, current status: {}",
                id,
                gstReportInfo.getStatus());

        throw new ReportingException(
                ErrorConstants.INVALID_REQUEST_ERROR_CODE,
                "GST report can be cancelled only when status is UPLOAD_SUCCESS");
    }

    gstReportInfoDao.updateGstStatusById(
            reportId,
            ReportStatus.CANCELLED.getName(),
            request.getRemark());

    log.info("GST report cancelled successfully for id: {}", id);
}


// ============================================================
// GstReportInfoDao.java
// ============================================================

public Optional<GstReportInfo> findById(UUID id) {
    return gstReportInfoRepository.findById(id);
}

@Transactional
public void updateGstStatusById(
        UUID id,
        String gstStatus,
        String remark) {

    gstReportInfoRepository.updateGstStatusById(
            id,
            gstStatus,
            remark);
}


// ============================================================
// GstReportInfoRepository.java
// ============================================================

@Modifying
@Transactional
@Query("""
        UPDATE GstReportInfo g
        SET g.status = :gstStatus,
            g.remark = :remark
        WHERE g.id = :id
        """)
void updateGstStatusById(
        @Param("id") UUID id,
        @Param("gstStatus") String gstStatus,
        @Param("remark") String remark);
============================================================
1. GstReportCancelDto.java
============================================================

package com.epay.reporting.dto;

import jakarta.validation.constraints.NotBlank;
import lombok.Data;

@Data
public class GstReportCancelDto {

    @NotBlank(message = "Remark is required")
    private String remark;
}


============================================================
2. GstReportController.java
============================================================

@PostMapping("/cancel/{id}")
public ReportingResponse<String> cancelGstReport(
        @PathVariable("id") String id,
        @Valid @RequestBody GstReportCancelDto request) {

    log.info(
            "Received GST report cancellation request for id: {}",
            id
    );

    gstReportInfoService.cancelGstReport(id, request);

    return ReportingResponse.<String>builder()
            .status(ResponseStatus.SUCCESS)
            .message("GST report cancelled successfully")
            .data("GST report cancelled successfully")
            .build();
}


============================================================
3. GstReportInfoService.java
============================================================

@Transactional
public void cancelGstReport(
        String id,
        GstReportCancelDto request) {

    log.info(
            "Cancelling GST report for id: {}",
            id
    );

    UUID reportId;

    try {

        reportId = UUID.fromString(id);

    } catch (IllegalArgumentException exception) {

        log.error(
                "Invalid GST report id: {}",
                id,
                exception
        );

        throw new ReportingException(
                "Invalid GST report id"
        );
    }

    GstReportInfo gstReportInfo =
            gstReportInfoDao.findById(reportId)
                    .orElseThrow(() ->
                            new ReportingException(
                                    "GST report not found for id: " + id
                            )
                    );

    log.info(
            "GST report found. Id: {}, Current status: {}",
            id,
            gstReportInfo.getStatus()
    );

    if (ReportStatus.CANCELLED.getName()
            .equals(gstReportInfo.getStatus())) {

        throw new ReportingException(
                "GST report is already cancelled"
        );
    }

    int updatedRecords =
            gstReportInfoDao.cancelGstReport(
                    reportId,
                    request.getRemark()
            );

    if (updatedRecords == 0) {

        throw new ReportingException(
                "GST report could not be cancelled"
        );
    }

    log.info(
            "GST report cancelled successfully. Id: {}",
            id
    );
}


============================================================
4. GstReportInfoDao.java
============================================================

public int cancelGstReport(
        UUID id,
        String remark) {

    log.info(
            "Cancelling GST report in database. Id: {}",
            id
    );

    int updatedRecords =
            gstReportInfoRepository.cancelGstReportById(
                    id,
                    ReportStatus.CANCELLED.getName(),
                    remark
            );

    log.info(
            "GST report cancellation completed. Records updated: {}",
            updatedRecords
    );

    return updatedRecords;
}


============================================================
5. GstReportInfoRepository.java
============================================================

@Modifying
@Transactional
@Query("""
        UPDATE GstReportInfo g
           SET g.status = :gstStatus,
               g.remark = :remark
         WHERE g.id = :id
        """)
int cancelGstReportById(
        @Param("id") UUID id,
        @Param("gstStatus") String gstStatus,
        @Param("remark") String remark
);


============================================================
6. API
============================================================

POST /report/v1/gst/cancel/{id}


============================================================
7. Request Body
============================================================

{
    "remark": "Acknowledgement not received within maximum days"
}


============================================================
8. Database Update
============================================================

status = CANCELLED
remark = <remark received from frontend>


============================================================
9. COMPLETE FLOW
============================================================

Frontend
    ↓
POST /report/v1/gst/cancel/{id}
    ↓
GstReportController
    ↓
cancelGstReport()
    ↓
GstReportInfoService
    ↓
cancelGstReport()
    ↓
GstReportInfoDao
    ↓
cancelGstReport()
    ↓
GstReportInfoRepository
    ↓
cancelGstReportById()
    ↓
DB
    ↓
STATUS = CANCELLED
REMARK = Frontend Remark





processGstReportAckCheck()




/**
 * Class Name: GstReportAckCheckScheduler
 *
 * Description: This class is responsible for scheduling and executing
 * tasks to check GST report acknowledgement status and update reports
 * when acknowledgement is not received within the configured maximum days.
 *
 * Author: Rohit Gardi
 * Copyright (c) 2026 [State Bank of India]
 * All rights reserved
 * Version: 1.0
 */




Scheduler:
checkGstReportAckScheduler()

Service:
processGstReportAckStatus()

DAO:
markAckNotReceivedReports()

Repository:
updateStatusAndRemarkForAckNotReceivedAfterMaxDays()







============================================================
1. application.yml
============================================================

scheduled:
  ack:
    max-days: 7
    cron: "0 0 * * * *"


============================================================
2. GstReportAckScheduler.java
============================================================

package com.epay.reporting.scheduler;

import com.epay.reporting.service.GstReportInfoService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
@RequiredArgsConstructor
@Slf4j
public class GstReportAckScheduler {

    private final GstReportInfoService gstReportInfoService;

    @Value("${scheduled.ack.max-days:7}")
    private int maxDays;

    @Scheduled(cron = "${scheduled.ack.cron}")
    public void markAckNotReceived() {

        log.info(
                "GST report ACK scheduler started. Max days: {}",
                maxDays
        );

        try {

            int updatedRecords =
                    gstReportInfoService.markAckNotReceived(maxDays);

            log.info(
                    "GST report ACK scheduler completed. Records updated: {}",
                    updatedRecords
            );

        } catch (Exception exception) {

            log.error(
                    "Error while marking GST reports as ACK_NOT_RECEIVED",
                    exception
            );
        }
    }
}


============================================================
3. GstReportInfoService.java
============================================================

/*
 * Existing GstReportInfoService class मध्ये
 * खालील method add कर.
 */

public int markAckNotReceived(int maxDays) {

    log.info(
            "Processing GST reports for ACK timeout. Max days: {}",
            maxDays
    );

    long cutoffTime = System.currentTimeMillis()
            - (maxDays * 24L * 60L * 60L * 1000L);

    log.info(
            "GST report ACK cutoff time: {}",
            cutoffTime
    );

    int updatedRecords =
            gstReportInfoDao.markAckNotReceived(cutoffTime);

    log.info(
            "GST report ACK timeout processing completed. Records updated: {}",
            updatedRecords
    );

    return updatedRecords;
}


============================================================
4. GstReportInfoDao.java
============================================================

/*
 * Existing GstReportInfoDao class मध्ये
 * खालील method add कर.
 */

public int markAckNotReceived(long cutoffTime) {

    log.info(
            "Marking GST reports as CANCELLED for push date before cutoff: {}",
            cutoffTime
    );

    int updatedRecords =
            gstReportInfoRepository.markAckNotReceived(
                    ReportStatus.CANCELLED,
                    cutoffTime
            );

    log.info(
            "GST reports marked as CANCELLED. Records updated: {}",
            updatedRecords
    );

    return updatedRecords;
}


============================================================
5. GstReportInfoRepository.java
============================================================

package com.epay.reporting.repository;

import com.epay.reporting.entity.GstReportInfo;
import com.epay.reporting.model.ReportStatus;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.transaction.annotation.Transactional;

import java.util.UUID;

public interface GstReportInfoRepository
        extends JpaRepository<GstReportInfo, UUID> {

    @Modifying
    @Transactional
    @Query("""
        UPDATE GstReportInfo g
           SET g.status = :cancelledStatus,
               g.remark = 'ACK_NOT_RECEIVED'
         WHERE g.status <> :cancelledStatus
           AND g.pushStatus = 'UPLOAD_SUCCESS'
           AND g.pushStatusDate <= :cutoffTime
        """)
    int markAckNotReceived(
            @Param("cancelledStatus") ReportStatus cancelledStatus,
            @Param("cutoffTime") long cutoffTime
    );
}


============================================================
6. FINAL BUSINESS LOGIC
============================================================

pushStatus = UPLOAD_SUCCESS
        +
pushStatusDate + 7 days <= current time
        +
status != CANCELLED
        ↓
status = CANCELLED
remark = ACK_NOT_RECEIVED


============================================================
7. EXAMPLE
============================================================

Push Date       : 24-Aug-2026
Max Days        : 7
Current Date    : 31-Aug-2026

7 days completed
        ↓
Scheduler checks record
        ↓
pushStatus = UPLOAD_SUCCESS
        ↓
pushStatusDate <= cutoffTime
        ↓
Status  = CANCELLED
Remark  = ACK_NOT_RECEIVED


============================================================
8. CRON
============================================================

0 0 * * * *

Scheduler प्रत्येक तासाला minute 00 ला चालेल.

01:00
02:00
03:00
...
23:00


============================================================
IMPORTANT
============================================================

DB मध्ये ackReceived field नाही.

म्हणून:

ackReceived = false

ही condition वापरलेली नाही.

ACK आला/नाही हे existing status/remark/business
flow वर depend असेल.

या scheduler मध्ये आपण फक्त:

1. SFTP push SUCCESS आहे का?
2. Push date पासून configured 7 days झाले का?
3. Record आधीच CANCELLED नाही ना?

हे check करतो.

Eligible record मिळाला तर:

STATUS = CANCELLED
REMARK = ACK_NOT_RECEIVED
