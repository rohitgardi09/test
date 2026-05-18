# FILE : application.yml

exception-tracker:
  retention-days: 7
  cleanup-cron: "0 0 2 * * ?"

=============================================================================

// FILE : ExceptionTrackerQuery.java

package com.sbi.epay.exceptionTracker.query;

/**
 * Class Name : ExceptionTrackerQuery
 * Description : Query class used to store exception tracker SQL queries.
 * Author : V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 */

public class ExceptionTrackerQuery {

    private ExceptionTrackerQuery() {
    }

    /**
     * Query used to drop old Oracle partition
     */
    public static final String DROP_EXCEPTION_LOG_PARTITION =
            """
            ALTER TABLE EXCEPTION_LOG
            DROP PARTITION FOR (
                TO_DATE('%s','YYYY-MM-DD')
            )
            """;
}

=============================================================================

// FILE : ExceptionPartitionCleanupService.java

package com.sbi.epay.exceptionTracker.service;

import com.sbi.epay.exceptionTracker.query.ExceptionTrackerQuery;

import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;

import lombok.RequiredArgsConstructor;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;

/**
 * Class Name : ExceptionPartitionCleanupService
 * Description : Service class used to drop old exception log partitions.
 * Author : V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 */

@Service
@RequiredArgsConstructor
public class ExceptionPartitionCleanupService {

    private static final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(
                    ExceptionPartitionCleanupService.class);

    private final JdbcTemplate jdbcTemplate;

    /**
     * Drops old partition based on partition date
     */
    public void dropExceptionLogPartition(
            String partitionDate) {

        String sql = String.format(
                ExceptionTrackerQuery
                        .DROP_EXCEPTION_LOG_PARTITION,
                partitionDate
        );

        jdbcTemplate.execute(sql);

        logger.info(
                "Exception log partition dropped successfully for date : {}",
                partitionDate
        );
    }
}

=============================================================================

// FILE : ExceptionCleanupScheduler.java

package com.sbi.epay.exceptionTracker.scheduler;

import com.sbi.epay.exceptionTracker.service.ExceptionPartitionCleanupService;

import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;

import lombok.RequiredArgsConstructor;

import net.javacrumbs.shedlock.spring.annotation.SchedulerLock;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

/**
 * Class Name : ExceptionCleanupScheduler
 * Description : Scheduler class used to clean old exception log partitions.
 * Author : V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 */

@Component
@RequiredArgsConstructor
public class ExceptionCleanupScheduler {

    private static final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(
                    ExceptionCleanupScheduler.class);

    private final ExceptionPartitionCleanupService
            exceptionPartitionCleanupService;

    @Value("${exception-tracker.retention-days:7}")
    private int retentionDays;

    @Value("${exception-tracker.cleanup-cron:0 0 2 * * ?}")
    private String cleanupCron;

    /**
     * Scheduler runs based on configured cron expression
     * and drops old exception log partitions.
     */
    @Scheduled(
            cron =
                    "${exception-tracker.cleanup-cron:0 0 2 * * ?}"
    )
    @SchedulerLock(
            name = "dropExceptionLogPartition",
            lockAtLeastFor = "PT1M",
            lockAtMostFor = "PT10M"
    )
    public void dropOldPartition() {

        logger.info(
                "Exception cleanup scheduler started with cron : {}",
                cleanupCron
        );

        try {

            String partitionDate =
                    LocalDate.now()
                            .minusDays(retentionDays)
                            .format(
                                    DateTimeFormatter
                                            .ofPattern("yyyy-MM-dd")
                            );

            exceptionPartitionCleanupService
                    .dropExceptionLogPartition(
                            partitionDate
                    );

        } catch (Exception ex) {

            logger.error(
                    "Error while dropping exception log partition",
                    ex
            );
        }

        logger.info(
                "Exception cleanup scheduler completed"
        );
    }
}

###################



# FILE : application.yml

exception-tracker:
  retention-days: 7
  cleanup-cron: "0 0 2 * * ?"

=============================================================================

// FILE : ExceptionTrackerQuery.java

package com.sbi.epay.exceptionTracker.query;

/**
 * Class Name : ExceptionTrackerQuery
 * Description : Query class used to store exception tracker SQL queries.
 * Author : V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 */

public class ExceptionTrackerQuery {

    private ExceptionTrackerQuery() {
    }

    /**
     * Query used to drop old Oracle partition
     */
    public static final String DROP_EXCEPTION_LOG_PARTITION =
            """
            ALTER TABLE EXCEPTION_LOG
            DROP PARTITION FOR (
                TO_DATE('%s','YYYY-MM-DD')
            )
            """;
}

=============================================================================

// FILE : ExceptionPartitionCleanupService.java

package com.sbi.epay.exceptionTracker.service;

import com.sbi.epay.exceptionTracker.query.ExceptionTrackerQuery;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;

/**
 * Class Name : ExceptionPartitionCleanupService
 * Description : Service class used to drop old exception log partitions.
 * Author : V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 */

@Slf4j
@Service
@RequiredArgsConstructor
public class ExceptionPartitionCleanupService {

    private final JdbcTemplate jdbcTemplate;

    /**
     * Drops old partition based on partition date
     */
    public void dropExceptionLogPartition(
            String partitionDate) {

        String sql = String.format(
                ExceptionTrackerQuery
                        .DROP_EXCEPTION_LOG_PARTITION,
                partitionDate
        );

        jdbcTemplate.execute(sql);

        log.info(
                "Exception log partition dropped successfully for date : {}",
                partitionDate
        );
    }
}

=============================================================================

// FILE : ExceptionCleanupScheduler.java

package com.sbi.epay.exceptionTracker.scheduler;

import com.sbi.epay.exceptionTracker.service.ExceptionPartitionCleanupService;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import net.javacrumbs.shedlock.spring.annotation.SchedulerLock;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

/**
 * Class Name : ExceptionCleanupScheduler
 * Description : Scheduler class used to clean old exception log partitions.
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

    private final ExceptionPartitionCleanupService
            exceptionPartitionCleanupService;

    @Value("${exception-tracker.retention-days:7}")
    private int retentionDays;

    @Value("${exception-tracker.cleanup-cron:0 0 2 * * ?}")
    private String cleanupCron;

    /**
     * Scheduler runs based on configured cron expression
     * and drops old exception log partitions.
     */
    @Scheduled(
            cron =
                    "${exception-tracker.cleanup-cron:0 0 2 * * ?}"
    )
    @SchedulerLock(
            name = "dropExceptionLogPartition",
            lockAtLeastFor = "PT1M",
            lockAtMostFor = "PT10M"
    )
    public void dropOldPartition() {

        log.info(
                "Exception cleanup scheduler started with cron : {}",
                cleanupCron
        );

        try {

            String partitionDate =
                    LocalDate.now()
                            .minusDays(retentionDays)
                            .format(
                                    DateTimeFormatter
                                            .ofPattern("yyyy-MM-dd")
                            );

            exceptionPartitionCleanupService
                    .dropExceptionLogPartition(
                            partitionDate
                    );

        } catch (Exception ex) {

            log.error(
                    "Error while dropping exception log partition",
                    ex
            );
        }

        log.info(
                "Exception cleanup scheduler completed"
        );
    }
}


//////
# application.yml

exception-tracker:
  retention-days: 7
  cleanup-cron: "0 0 2 * * ?"

// package:
// com.sbi.epay.exceptionTracker.query

public class ExceptionTrackerQuery {

    private ExceptionTrackerQuery() {
    }

    public static final String DROP_EXCEPTION_LOG_PARTITION =
            """
            ALTER TABLE EXCEPTION_LOG
            DROP PARTITION FOR (
                TO_DATE('%s','YYYY-MM-DD')
            )
            """;
}

// package:
// com.sbi.epay.exceptionTracker.service

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
@Slf4j
public class ExceptionPartitionCleanupService {

    private final JdbcTemplate jdbcTemplate;

    public void dropExceptionLogPartition(String partitionDate) {

        String sql = String.format(
                ExceptionTrackerQuery.DROP_EXCEPTION_LOG_PARTITION,
                partitionDate
        );

        jdbcTemplate.execute(sql);

        log.info(
                "Exception log partition dropped successfully for date: {}",
                partitionDate
        );
    }
}

// package:
// com.sbi.epay.exceptionTracker.scheduler

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import net.javacrumbs.shedlock.spring.annotation.SchedulerLock;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

@Slf4j
@Component
@RequiredArgsConstructor
public class ExceptionCleanupScheduler {

    private final ExceptionPartitionCleanupService
            exceptionPartitionCleanupService;

    @Value("${exception-tracker.retention-days:7}")
    private int retentionDays;

    @Value("${exception-tracker.cleanup-cron:0 0 2 * * ?}")
    private String cleanupCron;

    @Scheduled(
            cron = "${exception-tracker.cleanup-cron:0 0 2 * * ?}"
    )
    @SchedulerLock(
            name = "dropExceptionLogPartition",
            lockAtLeastFor = "PT1M",
            lockAtMostFor = "PT10M"
    )
    public void dropOldPartition() {

        log.info(
                "Exception cleanup scheduler started with cron: {}",
                cleanupCron
        );

        try {

            String partitionDate = LocalDate.now()
                    .minusDays(retentionDays)
                    .format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));

            exceptionPartitionCleanupService
                    .dropExceptionLogPartition(partitionDate);

        } catch (Exception ex) {

            log.error(
                    "Error while dropping exception log partition",
                    ex
            );
        }

        log.info("Exception cleanup scheduler completed");
    }
}




CREATE TABLE EXCEPTION_LOG (

    ID NUMBER GENERATED BY DEFAULT AS IDENTITY,

    SERVICE_NAME VARCHAR2(50),

    CLASS_NAME VARCHAR2(100),

    METHOD_NAME VARCHAR2(100),

    EXCEPTION_TYPE VARCHAR2(100),

    EXCEPTION_MESSAGE CLOB,

    STACK_TRACE CLOB,

    MERCHANT_ID VARCHAR2(20),

    ORDER_REF_NUMBER VARCHAR2(50),

    ATRN_NUM VARCHAR2(50),

    PAY_MODE VARCHAR2(50),

    CORRELATION_ID VARCHAR2(50),

    REMARK VARCHAR2(500),

    MDC_JSON CLOB,

    CREATED_BY VARCHAR2(100),

    CREATED_DATE NUMBER,

    PARTITION_DATE DATE GENERATED ALWAYS AS
    (
        TO_DATE(
            '1970-01-01 00:00:00',
            'YYYY-MM-DD HH24:MI:SS'
        )
        + (CREATED_DATE / 1000 / 86400)
    ) VIRTUAL

)

PCTFREE 30
INITRANS 100
MAXTRANS 255

PARTITION BY RANGE (PARTITION_DATE)

INTERVAL
(
    NUMTODSINTERVAL(1, 'DAY')
)

(
    PARTITION P_START
    VALUES LESS THAN
    (
        TO_DATE(
            '2026-01-01 00:00:00',
            'YYYY-MM-DD HH24:MI:SS'
        )
    )
);

=============================================================================

package com.sbi.epay.exceptionTracker.scheduler;

import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

/**
 * Class Name : ExceptionCleanupScheduler
 * Description : Scheduler used to drop old Oracle partitions.
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

    private static final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(
                    ExceptionCleanupScheduler.class);

    private final JdbcTemplate jdbcTemplate;

    @Value("${exception-tracker.retention-days}")
    private int retentionDays;

    /**
     * Scheduler runs based on yml cron expression
     */
    @Scheduled(cron = "${exception-tracker.cleanup-cron}")
    public void dropOldPartition() {

        try {

            String partitionDate =
                    LocalDate.now()
                            .minusDays(retentionDays)
                            .format(
                                    DateTimeFormatter
                                            .ofPattern("yyyy-MM-dd"));

            String sql = String.format(
                    """
                    ALTER TABLE EXCEPTION_LOG
                    DROP PARTITION FOR (
                        TO_DATE('%s', 'YYYY-MM-DD')
                    )
                    """,
                    partitionDate);

            jdbcTemplate.execute(sql);

            logger.info(
                    "Dropped partition for date : {}",
                    partitionDate);

        } catch (Exception ex) {

            logger.error(
                    "Error while dropping old partition",
                    ex);
        }
    }
}

=============================================================================

exception-tracker:
  retention-days: 7
  cleanup-cron: "0 */5 * * * ?"

//////////

CREATE TABLE EXCEPTION_LOG (

ID                NUMBER GENERATED BY DEFAULT AS IDENTITY,

SERVICE_NAME      							 VARCHAR2(50),

CLASS_NAME        							VARCHAR2(100),

METHOD_NAME      						    VARCHAR2(100),

EXCEPTION_TYPE  						    VARCHAR2(100),

EXCEPTION_MESSAGE 									 CLOB,

STACK_TRACE       									 CLOB,

MERCHANT_ID      						     VARCHAR2(20),

ORDER_REF_NUMBER  							 VARCHAR2(50),

ATRN_NUM          							 VARCHAR2(50),

PAY_MODE          							 VARCHAR2(50),

CORRELATION_ID    							 VARCHAR2(50),

REMARK            							VARCHAR2(500),

MDC_JSON          									 CLOB,

CREATED_BY        							VARCHAR2(100),

CREATED_DATE      									NUMBER,

PARTITION_DATE DATE GENERATED ALWAYS AS (TO_DATE(' 1970-01-01 00:00:00', 'syyyy-mm-dd hh24:mi:ss')+CREATED_DATE/1000/86400) VIRTUAL 

)

PCTFREE 30 INITRANS 100 MAXTRANS 255

   PARTITION BY RANGE (PARTITION_DATE) INTERVAL (NUMTODSINTERVAL(1, 'DAY'))

  (PARTITION P_START  VALUES LESS THAN (TO_DATE(' 2026-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN')) SEGMENT CREATION DEFERRED

   PCTFREE 10  INITRANS 100);
 



# Exception Tracker Configuration

exception-tracker:
  retention-days: 7
  cleanup-cron: "0 */5 * * * ?"

////


package com.sbi.epay.exceptionTracker.scheduler;

import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

/**
 * Class Name : ExceptionCleanupScheduler
 * Description : Scheduler used to drop old Oracle partitions.
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

    private static final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(
                    ExceptionCleanupScheduler.class);

    private final JdbcTemplate jdbcTemplate;

    @Value("${exception-tracker.retention-days}")
    private int retentionDays;

    /**
     * Scheduler runs based on yml cron expression
     */
    @Scheduled(cron = "${exception-tracker.cleanup-cron}")
    public void dropOldPartition() {

        try {

            String partitionDate =
                    LocalDate.now()
                            .minusDays(retentionDays)
                            .format(
                                    DateTimeFormatter
                                            .ofPattern("yyyy-MM-dd"));

            String sql = String.format(
                    """
                    ALTER TABLE EXCEPTION_LOG
                    DROP PARTITION FOR (
                        TO_DATE('%s', 'yyyy-MM-dd')
                    )
                    """,
                    partitionDate);

            jdbcTemplate.execute(sql);

            logger.info(
                    "Dropped partition for date : {}",
                    partitionDate);

        } catch (Exception ex) {

            logger.error(
                    "Error while dropping old partition",
                    ex);
        }
    }
}


//////
INSERT INTO EXCEPTION_LOG (

    SERVICE_NAME,
    CLASS_NAME,
    METHOD_NAME,
    EXCEPTION_TYPE,
    EXCEPTION_MESSAGE,
    STACK_TRACE,
    CREATED_DATE

)
VALUES (

    'TEST_SERVICE',
    'TEST_CLASS',
    'TEST_METHOD',
    'java.lang.RuntimeException',
    'Test Exception',
    'Test Stack Trace',
    1746000000000

);

COMMIT;



PARTITION BY RANGE (CREATED_DATE)
INTERVAL (86400000)
(
    PARTITION P_INITIAL VALUES LESS THAN (0)
)



-- CHANGESET RohitG:1.1

ALTER TABLE EXCEPTION_LOG
PARTITION BY RANGE (CREATED_DATE)
INTERVAL (86400000)
(
    PARTITION P_INITIAL VALUES LESS THAN (0)
);





// FILE : ExceptionTestController.java

package com.sbi.epay.exceptionTracker.controller;

import com.sbi.epay.exceptionTracker.annotation.TrackException;

import org.slf4j.MDC;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

/**
 * Class Name: ExceptionTestController
 * Description: Test controller used to generate exceptions for testing.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@RestController
public class ExceptionTestController {

    /**
     * Test API to generate runtime exception.
     */
    @TrackException
    @GetMapping("/test/exception")
    public String testException() {

        MDC.put(
                "MID",
                UUID.randomUUID().toString());

        MDC.put(
                "ORDER_REF",
                UUID.randomUUID().toString());

        MDC.put(
                "ATRN",
                UUID.randomUUID().toString());

        MDC.put(
                "PAYMODE",
                "UPI");

        MDC.put(
                "CORRELATION_ID",
                UUID.randomUUID().toString());

        MDC.put(
                "REMARK",
                "Exception tracker testing");

        MDC.put(
                "CREATED_BY",
                "SYSTEM");

        throw new RuntimeException(
                "Test exception generated successfully");
    }
}



// FILE : ErrorConstant.java

package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

/**
 * Class Name: ErrorConstant
 * Description: Utility class used to store application constants.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@UtilityClass
public class ErrorConstant {

    public static final String MID = "MID";
    public static final String ORDER_REF = "ORDER_REF";
    public static final String ATRN = "ATRN";
    public static final String PAYMODE = "PAYMODE";
    public static final String CORRELATION_ID = "CORRELATION_ID";
    public static final String REMARK = "REMARK";
    public static final String CREATED_BY = "CREATED_BY";

    public static final int STACK_TRACE_LIMIT = 10;
    public static final int QUEUE_SIZE = 10000;
    public static final int BATCH_SIZE = 50;
    public static final int EXCEPTION_LOG_RETENTION_DAYS = 7;

    public static final String DEFAULT_EXCEPTION_MESSAGE =
            "Exception message is not available";

    public static final String DEFAULT_MDC_VALUE =
            "MDC value is not available";
}

=============================================================================

// FILE : ExceptionUtil.java

package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

/**
 * Class Name: ExceptionUtil
 * Description: Utility class used to extract exception message safely.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@UtilityClass
public class ExceptionUtil {

    public static String getMessage(Throwable ex) {

        if (ex == null) {
            return ErrorConstant.DEFAULT_EXCEPTION_MESSAGE;
        }

        return ex.getMessage() != null
                ? ex.getMessage()
                : ErrorConstant.DEFAULT_EXCEPTION_MESSAGE;
    }
}

=============================================================================

// FILE : MDCUtil.java

package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

import java.util.Map;

/**
 * Class Name: MDCUtil
 * Description: Utility class used to fetch MDC values safely.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@UtilityClass
public class MDCUtil {

    /**
     * Fetches MDC value ignoring key case
     */
    public static String getIgnoreCase(
            Map<String, String> map,
            String... keys) {

        if (map == null) {
            return ErrorConstant.DEFAULT_MDC_VALUE;
        }

        for (String key : keys) {

            for (Map.Entry<String, String> entry
                    : map.entrySet()) {

                if (entry.getKey().equalsIgnoreCase(key)) {

                    return entry.getValue() != null
                            ? entry.getValue()
                            : ErrorConstant.DEFAULT_MDC_VALUE;
                }
            }
        }

        return ErrorConstant.DEFAULT_MDC_VALUE;
    }
}

=============================================================================

// FILE : StackTraceUtil.java

package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

/**
 * Class Name: StackTraceUtil
 * Description: Utility class used to generate formatted stack trace string.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@UtilityClass
public class StackTraceUtil {

    /**
     * Returns shortened stack trace
     */
    public static String getShortStackTrace(Throwable ex) {

        StringBuilder sb = new StringBuilder();

        sb.append(
                ExceptionUtil.getMessage(ex))
                .append(" ");

        StackTraceElement[] elements =
                ex.getStackTrace();

        int limit = Math.min(
                elements.length,
                ErrorConstant.STACK_TRACE_LIMIT);

        for (int i = 0; i < limit; i++) {

            sb.append(elements[i].toString())
                    .append("\n");
        }

        return sb.toString();
    }
}

// FILE : TrackException.java

package com.sbi.epay.exceptionTracker.annotation;

import java.lang.annotation.*;

/**
 * Class Name: TrackException
 * Description: Annotation used to mark methods for exception tracking.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface TrackException {

    boolean storeStackTrace() default true;

    boolean rethrow() default true;
}

=============================================================================

// FILE : QueueConfig.java

package com.sbi.epay.exceptionTracker.config;

import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.util.ErrorConstant;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

/**
 * Class Name: QueueConfig
 * Description: Configuration class used to create exception processing queue bean.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@Configuration
public class QueueConfig {

    @Bean
    public BlockingQueue<ExceptionLogDto> exceptionQueue() {

        return new LinkedBlockingQueue<>(
                ErrorConstant.QUEUE_SIZE);
    }
}

=============================================================================

// FILE : ExceptionQueueService.java

package com.sbi.epay.exceptionTracker.service;

import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;

import lombok.RequiredArgsConstructor;

import org.springframework.stereotype.Service;

import java.util.concurrent.BlockingQueue;

/**
 * Class Name: ExceptionQueueService
 * Description: Service class used to add exception details into processing queue.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@Service
@RequiredArgsConstructor
public class ExceptionQueueService {

    private final BlockingQueue<ExceptionLogDto>
            exceptionQueue;

    public void add(ExceptionLogDto dto) {

        exceptionQueue.offer(dto);
    }
}

=============================================================================

// FILE : ExceptionLogMapper.java

package com.sbi.epay.exceptionTracker.mapper;

import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.entity.ExceptionLog;

import org.mapstruct.Mapper;
import org.mapstruct.Mapping;

/**
 * Class Name: ExceptionLogMapper
 * Description: Mapper interface used to convert ExceptionLogDto into ExceptionLog entity.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@Mapper(componentModel = "spring")
public interface ExceptionLogMapper {

    @Mapping(target = "id", ignore = true)
    ExceptionLog toEntity(ExceptionLogDto dto);
}

// FILE : ExceptionLogDto.java

package com.sbi.epay.exceptionTracker.dto;

import lombok.*;

import java.util.Map;

/**
 * Class Name: ExceptionLogDto
 * Description: DTO class used to transfer exception details between layers.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ExceptionLogDto {

    private String serviceName;
    private String className;
    private String methodName;
    private String exceptionType;
    private String exceptionMessage;
    private String stackTrace;
    private String merchantId;
    private String orderRefNumber;
    private String atrnNum;
    private String payMode;
    private String correlationId;
    private String remark;
    private String mdcJson;
    private String createdBy;
    private Long createdDate;

    private Map<String, String> mdcMap;
}

=============================================================================

// FILE : ExceptionLog.java

package com.sbi.epay.exceptionTracker.entity;

import jakarta.persistence.*;

import lombok.*;

/**
 * Class Name: ExceptionLog
 * Description: Entity class mapped with EXCEPTION_LOG database table.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@Entity
@Table(name = "EXCEPTION_LOG")
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ExceptionLog {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String serviceName;
    private String className;
    private String methodName;
    private String exceptionType;

    @Lob
    private String exceptionMessage;

    @Lob
    private String stackTrace;

    private String merchantId;
    private String orderRefNumber;
    private String atrnNum;
    private String payMode;
    private String correlationId;
    private String remark;

    @Lob
    private String mdcJson;

    private String createdBy;

    private Long createdDate;
}

=============================================================================

// FILE : ExceptionLogRepository.java

package com.sbi.epay.exceptionTracker.repository;

import com.sbi.epay.exceptionTracker.entity.ExceptionLog;

import org.springframework.data.jpa.repository.JpaRepository;

/**
 * Class Name: ExceptionLogRepository
 * Description: Repository interface used for ExceptionLog database operations.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

public interface ExceptionLogRepository
        extends JpaRepository<ExceptionLog, Long> {
}

=============================================================================

// FILE : ExceptionLogDao.java

package com.sbi.epay.exceptionTracker.dao;

import com.sbi.epay.exceptionTracker.entity.ExceptionLog;
import com.sbi.epay.exceptionTracker.repository.ExceptionLogRepository;

import lombok.RequiredArgsConstructor;

import org.springframework.stereotype.Component;

import java.util.List;

/**
 * Class Name: ExceptionLogDao
 * Description: DAO class used to perform database operations for exception logs.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@Component
@RequiredArgsConstructor
public class ExceptionLogDao {

    private final ExceptionLogRepository repository;

    public void saveAll(List<ExceptionLog> logs) {

        repository.saveAll(logs);
    }
}


// FILE : ExceptionTrackerAspect.java

package com.sbi.epay.exceptionTracker.aspect;

import com.sbi.epay.exceptionTracker.annotation.TrackException;
import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.service.ExceptionQueueService;
import com.sbi.epay.exceptionTracker.util.ExceptionUtil;
import com.sbi.epay.exceptionTracker.util.StackTraceUtil;

import lombok.RequiredArgsConstructor;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;

import org.slf4j.MDC;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.util.Map;

/**
 * Class Name: ExceptionTrackerAspect
 * Description: Aspect class used to capture and process exceptions.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@Aspect
@Component
@RequiredArgsConstructor
public class ExceptionTrackerAspect {

    private final ExceptionQueueService queueService;

    @Value("${spring.application.name}")
    private String serviceName;

    @Around("@annotation(trackException)")
    public Object trackException(
            ProceedingJoinPoint joinPoint,
            TrackException trackException)
            throws Throwable {

        try {

            return joinPoint.proceed();

        } catch (Exception ex) {

            MethodSignature signature =
                    (MethodSignature)
                            joinPoint.getSignature();

            Map<String, String> mdc =
                    MDC.getCopyOfContextMap();

            ExceptionLogDto dto =
                    ExceptionLogDto.builder()
                            .serviceName(serviceName)
                            .className(
                                    signature
                                            .getDeclaringTypeName())
                            .methodName(
                                    signature
                                            .getMethod()
                                            .getName())
                            .exceptionType(
                                    ex.getClass()
                                            .getName())
                            .exceptionMessage(
                                    ExceptionUtil
                                            .getMessage(ex))
                            .stackTrace(
                                    trackException
                                            .storeStackTrace()
                                            ? StackTraceUtil
                                            .getShortStackTrace(ex)
                                            : null)
                            .mdcMap(mdc)
                            .build();

            queueService.add(dto);

            if (trackException.rethrow()) {
                throw ex;
            }

            throw new RuntimeException(ex);
        }
    }
}

=============================================================================

// FILE : ExceptionConsumerService.java

package com.sbi.epay.exceptionTracker.service;

import com.fasterxml.jackson.databind.ObjectMapper;

import com.sbi.epay.exceptionTracker.dao.ExceptionLogDao;
import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.entity.ExceptionLog;
import com.sbi.epay.exceptionTracker.mapper.ExceptionLogMapper;
import com.sbi.epay.exceptionTracker.util.ErrorConstant;
import com.sbi.epay.exceptionTracker.util.MDCUtil;

import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;

import jakarta.annotation.PostConstruct;

import lombok.RequiredArgsConstructor;

import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.BlockingQueue;

/**
 * Class Name: ExceptionConsumerService
 * Description: Service class used to consume exception queue and save logs.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@Service
@RequiredArgsConstructor
public class ExceptionConsumerService {

    private static final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(
                    ExceptionConsumerService.class);

    private final BlockingQueue<ExceptionLogDto>
            exceptionQueue;

    private final ExceptionLogDao dao;

    private final ExceptionLogMapper mapper;

    private final ObjectMapper objectMapper;

    @PostConstruct
    public void init() {

        Thread thread =
                new Thread(this::consume);

        thread.setDaemon(true);

        thread.start();
    }

    public void consume() {

        while (true) {

            try {

                List<ExceptionLog> logs =
                        new ArrayList<>();

                ExceptionLogDto dto =
                        exceptionQueue.take();

                prepare(dto);

                logs.add(
                        mapper.toEntity(dto));

                List<ExceptionLogDto> temp =
                        new ArrayList<>();

                exceptionQueue.drainTo(
                        temp,
                        ErrorConstant.BATCH_SIZE);

                for (ExceptionLogDto data : temp) {

                    prepare(data);

                    logs.add(
                            mapper.toEntity(data));
                }

                dao.saveAll(logs);

            } catch (Exception ex) {

                logger.error(
                        "Error while saving exception logs",
                        ex);
            }
        }
    }

    private void prepare(ExceptionLogDto dto)
            throws Exception {

        Map<String, String> mdc =
                dto.getMdcMap();

        dto.setMerchantId(
                MDCUtil.getIgnoreCase(
                        mdc,
                        ErrorConstant.MID));

        dto.setOrderRefNumber(
                MDCUtil.getIgnoreCase(
                        mdc,
                        ErrorConstant.ORDER_REF));

        dto.setAtrnNum(
                MDCUtil.getIgnoreCase(
                        mdc,
                        ErrorConstant.ATRN));

        dto.setPayMode(
                MDCUtil.getIgnoreCase(
                        mdc,
                        ErrorConstant.PAYMODE));

        dto.setCorrelationId(
                MDCUtil.getIgnoreCase(
                        mdc,
                        ErrorConstant.CORRELATION_ID));

        dto.setRemark(
                MDCUtil.getIgnoreCase(
                        mdc,
                        ErrorConstant.REMARK));

        dto.setCreatedBy(
                MDCUtil.getIgnoreCase(
                        mdc,
                        ErrorConstant.CREATED_BY));

        dto.setCreatedDate(
                System.currentTimeMillis());

        dto.setMdcJson(
                objectMapper.writeValueAsString(mdc));
    }
}


// FILE : ExceptionCleanupScheduler.java

package com.sbi.epay.exceptionTracker.scheduler;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

/**
 * Class Name : ExceptionCleanupScheduler
 * Description : Scheduler used to drop old Oracle partitions.
 * Author : V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 **/

@Slf4j
@Component
@RequiredArgsConstructor
public class ExceptionCleanupScheduler {

    private static final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(
                    ExceptionCleanupScheduler.class);

    private final JdbcTemplate jdbcTemplate;

    /**
     * Runs every day at 2 AM
     * Drops partition older than 7 days
     */
    @Scheduled(cron = "0 0 2 * * ?")
    public void dropOldPartition() {

        try {

            String partitionDate =
                    LocalDate.now()
                            .minusDays(
                                    ErrorConstant
                                            .EXCEPTION_LOG_RETENTION_DAYS)
                            .format(
                                    DateTimeFormatter
                                            .ofPattern("yyyy-MM-dd"));

            String sql = String.format("""
                    ALTER TABLE EXCEPTION_LOG
                    DROP PARTITION FOR (
                        TO_DATE('%s', 'YYYY-MM-DD')
                    )
                    """, partitionDate);

            jdbcTemplate.execute(sql);

            logger.info(
                    "Dropped partition for date : {}",
                    partitionDate);

        } catch (Exception ex) {

            logger.error(
                    "Error while dropping old partition",
                    ex);
        }
    }
}

=============================================================================

-- FILE : EXCEPTION_LOG.sql

CREATE TABLE EXCEPTION_LOG (

    ID NUMBER GENERATED BY DEFAULT AS IDENTITY,

    SERVICE_NAME VARCHAR2(50),

    CLASS_NAME VARCHAR2(100),

    METHOD_NAME VARCHAR2(100),

    EXCEPTION_TYPE VARCHAR2(100),

    EXCEPTION_MESSAGE CLOB,

    STACK_TRACE CLOB,

    MERCHANT_ID VARCHAR2(20),

    ORDER_REF_NUMBER VARCHAR2(50),

    ATRN_NUM VARCHAR2(50),

    PAY_MODE VARCHAR2(50),

    CORRELATION_ID VARCHAR2(50),

    REMARK VARCHAR2(500),

    MDC_JSON CLOB,

    CREATED_BY VARCHAR2(100),

    CREATED_DATE NUMBER

)
PARTITION BY RANGE (CREATED_DATE)

INTERVAL (86400000)

(
    PARTITION P_INITIAL VALUES LESS THAN (0)
);




