package com.epay.reporting.scheduler;

import com.epay.reporting.service.S3UploadRetryService;
import com.sbi.epay.authentication.util.EPayAuthenticationConstant;
import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;
import lombok.RequiredArgsConstructor;
import net.javacrumbs.shedlock.spring.annotation.SchedulerLock;
import org.slf4j.MDC;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.util.UUID;

/**
 * Class Name: S3UploadRetryScheduler
 * Description: This class is responsible for scheduling and re-attempting to upload the generated file to the S3 from the mount path.
 * Author: Akshaya Sahasranamam
 * Copyright (c) 2026 [State Bank of India]
 * All rights reserved
 * Version: 1.0
 */
@Component
@RequiredArgsConstructor
public class S3UploadRetryScheduler {

    private final S3UploadRetryService s3UploadRetryService;

    private final LoggerUtility log = LoggerFactoryUtility.getLogger(this.getClass());

    @Scheduled(cron = "${scheduled.report.upload.retry}")
    @SchedulerLock(name = "Report_upload_Scheduler", lockAtLeastFor = "PT30S", lockAtMostFor = "PT2M")
    public void s3UploadRetryScheduler() {
        MDC.put(EPayAuthenticationConstant.CORRELATION_ID, String.valueOf(UUID.randomUUID()));
        MDC.put(EPayAuthenticationConstant.SCENARIO, "RetryS3Upload");
        MDC.put(EPayAuthenticationConstant.OPERATION, "RetryS3Upload");
        log.info("Scheduler called for S3 upload retry");
        s3UploadRetryService.uploadReportsIntoS3();
        log.info("Scheduler completed for S3 upload retry");
    }
}
