.ExceptionConsumerService;
import com.sbi.epay.exceptionTracker.util.ErrorConstant;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.time.ZoneId;

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

    private final ExceptionLogDao exceptionLogDao;
    private static final LoggerUtility logger = LoggerFactoryUtility.getLogger(ExceptionConsumerService.class);


    /**
     * Runs every day at 2 AM
     * Deletes records older than configured retention days
     */
    @Scheduled(cron = "0 0 2 * * ?")
    public void deleteOldExceptionLogs() {
        logger.info("Exception cleanup scheduler started");
        long cutoffDate =LocalDateTime.now()
                .minusDays(ErrorConstant.EXCEPTION_LOG_RETENTION_DAYS)
                .atZone(ZoneId.systemDefault())
                .toInstant()
                .toEpochMilli();
        logger.info("Deleted exception logs ", exceptionLogDao.deleteByCreatedDateBefore(cutoffDate));
    }
}

=============================================================================
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
 * Description: Service class used to consume exception queue and save logs into database.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@Service
@RequiredArgsConstructor
public class ExceptionConsumerService {

    private static final LoggerUtility logger = LoggerFactoryUtility.getLogger(ExceptionConsumerService.class);

    private final BlockingQueue<ExceptionLogDto> exceptionQueue;
    private final ExceptionLogDao dao;
    private final ExceptionLogMapper mapper;
    private final ObjectMapper objectMapper;

    // Starts background consumer thread after application startup
    @PostConstruct
    public void init() {

        Thread thread = new Thread(this::consume);
        thread.setDaemon(true);
        thread.start();
    }

    public void consume() {

        // Continuously listens for exception logs from queue
        while (true) {
            try {
                List<ExceptionLog> logs = new ArrayList<>();

                // Waits until queue receives at least one log
                ExceptionLogDto dto = exceptionQueue.take();
                prepare(dto);
                logs.add(mapper.toEntity(dto));
                List<ExceptionLogDto> temp = new ArrayList<>();

                // Batch Size : 50 used to reduce database calls and improve performance.
                // Fetches remaining logs in batch to reduce DB calls
                exceptionQueue.drainTo(temp, ErrorConstant.BATCH_SIZE);

                for (ExceptionLogDto data : temp) {
                    prepare(data);
                    logs.add(mapper.toEntity(data));
                }
                // Saves logs into database using bulk insert
                dao.saveAll(logs);

            } catch (Exception ex) {
                logger.error("Error while saving exception logs", ex);
            }
        }
    }

    // Maps MDC values into DTO before DB save
    private void prepare(ExceptionLogDto dto)
            throws Exception {

        Map<String, String> mdc = dto.getMdcMap();
        dto.setMerchantId(MDCUtil.getIgnoreCase(mdc, ErrorConstant.MID));
        dto.setOrderRefNumber(MDCUtil.getIgnoreCase(mdc, ErrorConstant.ORDER_REF));
        dto.setAtrnNum(MDCUtil.getIgnoreCase(mdc, ErrorConstant.ATRN));
        dto.setPayMode(MDCUtil.getIgnoreCase(mdc, ErrorConstant.PAYMODE));
        dto.setCorrelationId(MDCUtil.getIgnoreCase(mdc, ErrorConstant.CORRELATION_ID));
        dto.setRemark(MDCUtil.getIgnoreCase(mdc, ErrorConstant.REMARK));
        dto.setCreatedBy(MDCUtil.getIgnoreCase(mdc, ErrorConstant.CREATED_BY));
        dto.setCreatedDate(System.currentTimeMillis());
        // Converts MDC map into JSON string
        dto.setMdcJson(objectMapper.writeValueAsString(mdc));
    }
}


=============================================================================

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
 * <p>
 * Version:1.0
 **/
@Service
@RequiredArgsConstructor
public class ExceptionQueueService {

    private final BlockingQueue<ExceptionLogDto> exceptionQueue;

    public void add(ExceptionLogDto dto) {
        exceptionQueue.offer(dto);
    }
}



=============================================================================
package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

/**
 * Class Name: ErrorConstant
 * Description: Utility class used to store application constants.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
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
    public static final String DEFAULT_EXCEPTION_MESSAGE = "Exception message is not available";
    public static final String DEFAULT_MDC_VALUE = "MDC value is not available";
}


=============================================================================
package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

/**
 * Class Name: ExceptionUtil
 * Description: Utility class used to extract exception message safely.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@UtilityClass
public class ExceptionUtil {

    public static String getMessage(Throwable ex) {

        if (ex == null) {
            return ErrorConstant.DEFAULT_EXCEPTION_MESSAGE;
        }
        return ex.getMessage() != null ? ex.getMessage() : ErrorConstant.DEFAULT_EXCEPTION_MESSAGE;
    }
}


=============================================================================
package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

import java.util.Map;

/**
 * Class Name: MDCUtil
 * Description: Utility class used to fetch MDC values safely.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@UtilityClass
public class MDCUtil {

    // Fetches MDC value ignoring key case
    public static String getIgnoreCase(Map<String, String> map, String... keys) {

        if (map == null) {
            return ErrorConstant.DEFAULT_MDC_VALUE;
        }
        for (String key : keys) {
            for (Map.Entry<String, String> entry : map.entrySet()) {
                if (entry.getKey().equalsIgnoreCase(key)) {
                    return entry.getValue() != null ? entry.getValue() : ErrorConstant.DEFAULT_MDC_VALUE;
                }
            }
        }
        return ErrorConstant.DEFAULT_MDC_VALUE;
    }
}


=============================================================================
package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

/**
 * Class Name: StackTraceUtil
 * Description: Utility class used to generate formatted stack trace string from exception.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@UtilityClass
public class StackTraceUtil {

    // Returns shortened stack trace
    public static String getShortStackTrace(Throwable ex) {

        StringBuilder sb = new StringBuilder();
        sb.append(ex.getMessage()).append(" ");
        StackTraceElement[] elements = ex.getStackTrace();

        // Stack Trace Limit : 10 used to avoid storing huge stack trace payload.
        int limit = Math.min(elements.length, ErrorConstant.STACK_TRACE_LIMIT);
        for (int i = 0; i < limit; i++) {
            sb
