File Name

src/main/java/com/epay/transaction/scheduler/ExceptionCleanupScheduler.java

package com.epay.transaction.scheduler;

import com.epay.transaction.entity.PaymentExceptionLog;
import com.epay.transaction.repository.ExceptionCleanupSchedulerRepository;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.util.List;

/**
 * Class Name: ExceptionCleanupScheduler
 * Description: Scheduler class used to delete old exception logs.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@Slf4j
@Component
@RequiredArgsConstructor
public class ExceptionCleanupScheduler {

    private final ExceptionCleanupSchedulerRepository
            repository;

    // Runs daily at 2 AM
    @Scheduled(cron = "0 0 2 * * ?")
    public void cleanupOldLogs() {

        // Calculates date before 7 days
        LocalDateTime cutoffDate =
                LocalDateTime.now()
                        .minusDays(7);

        // Fetches all exception logs
        List<PaymentExceptionLog> logs =
                repository.findAll();

        // Filters logs older than 7 days
        List<PaymentExceptionLog> oldLogs =
                logs.stream()
                        .filter(log ->
                                log.getCreatedDate()
                                        != null)
                        .filter(log ->
                                log.getCreatedDate()
                                        .isBefore(cutoffDate))
                        .toList();

        // Deletes old logs
        repository.deleteAll(oldLogs);

        log.info(
                "Deleted {} old exception logs",
                oldLogs.size());
    }
}

---

File Name

src/main/java/com/epay/transaction/repository/ExceptionCleanupSchedulerRepository.java

package com.epay.transaction.repository;

import com.epay.transaction.entity.PaymentExceptionLog;

import org.springframework.data.jpa.repository.JpaRepository;

/**
 * Class Name: ExceptionCleanupSchedulerRepository
 * Description: Repository interface used for scheduler cleanup DB operations.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
public interface ExceptionCleanupSchedulerRepository
        extends JpaRepository<
        PaymentExceptionLog,
        Long> {
}
