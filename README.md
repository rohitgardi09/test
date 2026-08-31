package com.epay.reporting.scheduler;

import com.epay.reporting.service.GstReportInfoService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Slf4j
@Component
@RequiredArgsConstructor
public class GstReportAckScheduler {

    private final GstReportInfoService gstReportInfoService;

    /**
     * Scheduler runs every hour to mark GST report records
     * where acknowledgement is not received within configured days.
     */
    @Scheduled(cron = "${scheduled.cron.gst-report.ack.cron}")
    public void markAckNotReceived() {

        log.info("GST Report ACK scheduler started");

        try {
            gstReportInfoService.markAckNotReceived();

            log.info("GST Report ACK scheduler completed successfully");
        } catch (Exception exception) {
            log.error(
                    "Error occurred while executing GST Report ACK scheduler",
                    exception
            );
        }
    }
}





***

@Value("${scheduled.cron.gst-report.ack.max-days}")
private int ackMaxDays;

/**
 * Marks GST report records as ACK_NOT_RECEIVED when
 * acknowledgement is not received within configured days.
 */
@Transactional
public void markAckNotReceived() {

    log.info(
            "Processing GST report records for ACK not received after {} days",
            ackMaxDays
    );

    long cutoffTime = System.currentTimeMillis()
            - (ackMaxDays * 24L * 60L * 60L * 1000L);

    int updatedRecords =
            gstReportInfoDao.markAckNotReceived(cutoffTime);

    log.info(
            "GST Report ACK processing completed. Updated records: {}",
            updatedRecords
    );
}





/**
 * Marks GST report records as ACK_NOT_RECEIVED when
 * acknowledgement is not received within configured days.
 *
 * @param cutoffTime cutoff time in milliseconds
 * @return number of records updated
 */
public int markAckNotReceived(long cutoffTime) {

    return gstReportInfoRepository.markAckNotReceived(cutoffTime);
}




package com.epay.reporting.repository;

import com.epay.reporting.entity.GstReportInfo;
import com.epay.reporting.model.ReportStatus;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.time.LocalDateTime;
import java.util.UUID;

public interface GstReportInfoRepository
        extends JpaRepository<GstReportInfo, UUID> {

    @Modifying
    @Query("""
            UPDATE GstReportInfo g
               SET g.status = :ackNotReceivedStatus
             WHERE g.status = :currentStatus
               AND g.pushStatusDate <= :cutoffDate
            """)
    int markAckNotReceived(
            @Param("currentStatus") ReportStatus currentStatus,
            @Param("ackNotReceivedStatus") ReportStatus ackNotReceivedStatus,
            @Param("cutoffDate") LocalDateTime cutoffDate
    );
}


