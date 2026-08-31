// ============================================================
// FILE 1
// Path:
// src/main/java/com/epay/reporting/scheduler/GstReportAckScheduler.java
// ============================================================

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

    @Value("${gst-report.ack.max-days}")
    private int maxDays;

    @Scheduled(cron = "${gst-report.ack.scheduler.cron}")
    public void markAckNotReceived() {

        log.info(
                "GST report ACK scheduler started for records older than {} days",
                maxDays);

        try {
            int updatedRecords =
                    gstReportInfoService.markAckNotReceived(maxDays);

            log.info(
                    "GST report ACK scheduler completed. Records updated: {}",
                    updatedRecords);

        } catch (Exception exception) {
            log.error(
                    "Error while marking GST reports as ACK_NOT_RECEIVED",
                    exception);
        }
    }
}


// ============================================================
// FILE 2
// Existing file:
// src/main/java/com/epay/reporting/service/GstReportInfoService.java
//
// Existing class मध्ये खालील method add कर.
// ============================================================

/*

import java.time.LocalDateTime;

*/

public int markAckNotReceived(int maxDays) {

    log.info(
            "Processing GST reports where ACK is not received within {} days",
            maxDays);

    LocalDateTime cutoffDate =
            LocalDateTime.now().minusDays(maxDays);

    int updatedRecords =
            gstReportInfoDao.markAckNotReceived(cutoffDate);

    log.info(
            "GST report ACK_NOT_RECEIVED processing completed. Records updated: {}",
            updatedRecords);

    return updatedRecords;
}


// ============================================================
// FILE 3
// Existing file:
// src/main/java/com/epay/reporting/dao/GstReportInfoDao.java
//
// Existing class मध्ये खालील method add कर.
// ============================================================

/*

import java.time.LocalDateTime;
import com.epay.reporting.model.ReportStatus;

*/

/**
 * Marks GST report records as CANCELLED and updates the remark
 * as ACK_NOT_RECEIVED when acknowledgement is not received
 * within configured maximum days.
 *
 * @param cutoffDate cutoff date for acknowledgement
 * @return number of records updated
 */
public int markAckNotReceived(LocalDateTime cutoffDate) {

    log.info(
            "Marking GST report records as CANCELLED where ACK is not received before {}",
            cutoffDate);

    return gstReportInfoRepository.markAckNotReceived(
            ReportStatus.CANCELLED,
            cutoffDate);
}


// ============================================================
// FILE 4
// Path:
// src/main/java/com/epay/reporting/repository/GstReportInfoRepository.java
// ============================================================

package com.epay.reporting.repository;

import com.epay.reporting.entity.GstReportInfo;
import com.epay.reporting.model.ReportStatus;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDateTime;
import java.util.UUID;

public interface GstReportInfoRepository
        extends JpaRepository<GstReportInfo, UUID> {

    /**
     * Marks GST report records as CANCELLED and updates the
     * remark as ACK_NOT_RECEIVED when acknowledgement is not
     * received within configured maximum days.
     *
     * @param cancelledStatus cancelled report status
     * @param cutoffDate cutoff date for acknowledgement
     * @return number of records updated
     */
    @Modifying
    @Transactional
    @Query("""
            UPDATE GstReportInfo g
               SET g.status = :cancelledStatus,
                   g.remark = 'ACK_NOT_RECEIVED'
             WHERE g.status <> :cancelledStatus
               AND g.pushStatus = 'SUCCESS'
               AND g.pushStatusDate <= :cutoffDate
            """)
    int markAckNotReceived(
            @Param("cancelledStatus") ReportStatus cancelledStatus,
            @Param("cutoffDate") LocalDateTime cutoffDate
    );
}


// ============================================================
// FILE 5
// resources/application.yml
//
// Existing application.yml मध्ये हा configuration add कर.
// ============================================================

gst-report:
  ack:
    max-days: 7
    scheduler:
      cron: "0 0 * * * *"




      ######


      हो 👍 सगळं एकाच ठिकाणी copy-paste करता येईल असं देतो. फक्त respective files मध्ये respective code टाक.

# ============================================================
# application.yml
# ============================================================

gst-report:
  ack:
    max-days: 7
    scheduler:
      cron: "0 0 * * * *"

// ============================================================
// GstReportAckScheduler.java
// ============================================================

package com.epay.reporting.scheduler;

import com.epay.reporting.service.GstReportInfoService;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
@RequiredArgsConstructor
public class GstReportAckScheduler {

    private final GstReportInfoService gstReportInfoService;

    @Value("${gst-report.ack.max-days}")
    private int maxDays;

    @Scheduled(cron = "${gst-report.ack.scheduler.cron}")
    public void markAckNotReceived() {

        gstReportInfoService.markAckNotReceived(maxDays);
    }
}

// ============================================================
// GstReportInfoService.java
// ============================================================

public int markAckNotReceived(int maxDays) {

    long cutoffTime = System.currentTimeMillis()
            - (maxDays * 24L * 60L * 60L * 1000L);

    return gstReportInfoRepository.markAckNotReceived(
            ReportStatus.CANCELLED,
            cutoffTime
    );
}

// ============================================================
// GstReportInfoRepository.java
// ============================================================

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
           AND g.ackReceived = false
           AND g.pushStatusDate <= :cutoffTime
        """)
    int markAckNotReceived(
            @Param("cancelledStatus") ReportStatus cancelledStatus,
            @Param("cutoffTime") long cutoffTime
    );
}

Flow: pushStatusDate पासून 7 दिवस पूर्ण → ackReceived = false → status = CANCELLED आणि remark = ACK_NOT_RECEIVED.

cron: "0 0 * * * *" मुळे हे दर तासाला 00 मिनिटाला check होईल.
