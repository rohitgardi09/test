Updated File Names

1. src/main/java/com/epay/transaction/scheduler/ExceptionCleanupScheduler.java

package com.epay.transaction.scheduler;

import com.epay.transaction.entity.PaymentExceptionLog;
import com.epay.transaction.repository.ExceptionCleanupSchedulerRepository;
import com.epay.transaction.util.ErrorConstant;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.util.List;

/**
 * Class Name: ExceptionCleanupScheduler
 * Description: Scheduler class used to delete old exception logs in batch.
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

    private final ExceptionCleanupSchedulerRepository repository;

    // Runs daily at 2 AM
    @Scheduled(cron = "0 0 2 * * ?")
    public void cleanupOldLogs() {

        // Deletes logs older than 7 days
        LocalDateTime cutoffDate =
                LocalDateTime.now()
                        .minusDays(7);

        Pageable pageable =
                PageRequest.of(
                        0,
                        ErrorConstant.BATCH_SIZE);

        while (true) {

            List<PaymentExceptionLog> logs =
                    repository.findAll(pageable)
                            .getContent();

            List<PaymentExceptionLog> oldLogs =
                    logs.stream()
                            .filter(log ->
                                    log.getCreatedDate()
                                            != null)
                            .filter(log ->
                                    log.getCreatedDate()
                                            .isBefore(cutoffDate))
                            .toList();

            if (oldLogs.isEmpty()) {

                break;
            }

            repository.deleteAllInBatch(
                    oldLogs);

            log.info(
                    "Deleted {} old exception logs",
                    oldLogs.size());
        }

        log.info(
                "Exception log cleanup completed");
    }
}

---

2. src/main/java/com/epay/transaction/repository/ExceptionCleanupSchedulerRepository.java

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

---

3. src/main/java/com/epay/transaction/util/ErrorConstant.java

package com.epay.transaction.util;

/**
 * Class Name: ErrorConstant
 * Description: Constant class used for exception tracker utility constants.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
public class ErrorConstant {

    private ErrorConstant() {
    }

    public static final int BATCH_SIZE =
            1000;
}
