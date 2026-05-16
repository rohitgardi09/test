// ======================================================
// FILE : Constant.java
// PACKAGE : com.sbi.epay.exceptionTracker.util
// ======================================================

package com.sbi.epay.exceptionTracker.util;

public final class Constant {

    private Constant() {
    }

    public static final int EXCEPTION_LOG_RETENTION_DAYS = 7;
}


// ======================================================
// FILE : ExceptionCleanupSchedulerRepository.java
// PACKAGE : com.sbi.epay.exceptionTracker.repository
// ======================================================

package com.sbi.epay.exceptionTracker.repository;

import com.sbi.epay.exceptionTracker.entity.ExceptionLog;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.time.LocalDateTime;

@Repository
public interface ExceptionCleanupSchedulerRepository
        extends JpaRepository<ExceptionLog, Long> {

    int deleteByCreatedDateBefore(LocalDateTime cutoffDate);
}


// ======================================================
// FILE : ExceptionCleanupScheduler.java
// PACKAGE : com.sbi.epay.exceptionTracker.scheduler
// ======================================================

package com.sbi.epay.exceptionTracker.scheduler;

import com.sbi.epay.exceptionTracker.repository.ExceptionCleanupSchedulerRepository;
import com.sbi.epay.exceptionTracker.util.Constant;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

/**
 * Class Name : ExceptionCleanupScheduler
 * Description : Scheduler used to delete old exception logs from database.
 * Author : V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 */

@Slf4j
@Component
@RequiredArgsConstructor
public class ExceptionCleanupScheduler {

    private final ExceptionCleanupSchedulerRepository repository;

    /**
     * Runs every day at 2 AM
     * Deletes records older than configured retention days
     */
    @Scheduled(cron = "0 0 2 * * ?")
    public void deleteOldExceptionLogs() {

        log.info("Exception cleanup scheduler started");

        LocalDateTime cutoffDate =
                LocalDateTime.now()
                        .minusDays(Constant.EXCEPTION_LOG_RETENTION_DAYS);

        int deletedCount =
                repository.deleteByCreatedDateBefore(cutoffDate);

        log.info("Deleted exception logs count : {}", deletedCount);

        log.info("Exception cleanup scheduler completed");
    }
}
