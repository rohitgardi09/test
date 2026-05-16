exceptionTracker
│
├── annotation
│   └── TrackException.java
│
├── aspect
│   └── ExceptionTrackerAspect.java
│
├── config
│   ├── QueueConfig.java
│   └── SchedulerConfig.java
│
├── controller
│   └── TestController.java
│
├── dao
│   └── ExceptionLogDao.java
│
├── dto
│   ├── BaseExceptionDto.java
│   └── PaymentExceptionDto.java
│
├── entity
│   ├── BaseExceptionEntity.java
│   └── PaymentExceptionLog.java
│
├── factory
│   └── ExceptionProcessorFactory.java
│
├── mapper
│   └── ExceptionLogMapper.java
│
├── processor
│   ├── ExceptionProcessor.java
│   └── PaymentExceptionProcessor.java
│
├── repository
│   └── ExceptionLogRepository.java
│
├── service
│   ├── ExceptionCleanupScheduler.java
│   ├── ExceptionConsumerService.java
│   └── ExceptionQueueService.java
│
└── util
    ├── ErrorConstant.java
    ├── ExceptionUtil.java
    ├── MDCUtil.java
    └── StackTraceUtil.java




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
 * <p>
 * Version:1.0
 **/
@Mapper(componentModel = "spring")
public interface ExceptionLogMapper {

    // Ignore id because database generates it automatically
    @Mapping(
            target = "id",
            ignore = true)
    ExceptionLog toEntity(
            ExceptionLogDto dto);
}

========================================================

package com.sbi.epay.exceptionTracker.controller;

import com.sbi.epay.exceptionTracker.annotation.TrackException;

import org.slf4j.MDC;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * Class Name: TestController
 * Description: Test controller used to validate exception tracking utility flow.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@RestController
public class TestController {

    @GetMapping("/test-exception")
    @TrackException
    public String testException() {

        // Adding sample MDC values for testing
        MDC.put(
                "MID",
                "MID12345");

        MDC.put(
                "ORDER_REF",
                "ORD12345");

        MDC.put(
                "ATRN",
                "ATRN12345");

        MDC.put(
                "PAYMODE",
                "UPI");

        MDC.put(
                "CORRELATION_ID",
                "CORR12345");

        MDC.put(
                "REMARK",
                "Test Exception");

        MDC.put(
                "CREATED_BY",
                "ROHIT");

        // Test exception
        throw new RuntimeException(
                "Test exception generated");
    }
}



git config --global user.name
git config --global user.email

git config --global user.name "Rohit Gardi"
git config --global user.email "tuzamail@example.com"




Exception Tracker - Complete Files
src/main/java/com/sbi/epay/exceptionTracker
│
├── annotation
│   └── TrackException.java
│
├── aspect
│   └── ExceptionTrackerAspect.java
│
├── config
│   └── QueueConfig.java
│
├── dao
│   └── ExceptionLogDao.java
│
├── dto
│   └── ExceptionLogDto.java
│
├── entity
│   └── ExceptionLog.java
│
├── mapper
│   └── ExceptionLogMapper.java
│
├── repository
│   └── ExceptionLogRepository.java
│
├── service
│   ├── ExceptionConsumerService.java
│   └── ExceptionQueueService.java
│
└── util
    ├── ErrorConstant.java
    ├── ExceptionUtil.java
    ├── MDCUtil.java
    └── StackTraceUtil.java
________________________________________
1.TrackException.java
package com.sbi.epay.exceptionTracker.annotation;

import java.lang.annotation.*;

/**
 * Class Name: TrackException
 * Description: Annotation used to mark methods for exception tracking.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface TrackException {

    boolean storeStackTrace() default true;

    boolean rethrow() default true;
}
________________________________________
2.ErrorConstant.java
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

    public static final String ORDER_REF =
            "ORDER_REF";

    public static final String ATRN =
            "ATRN";

    public static final String PAYMODE =
            "PAYMODE";

    public static final String CORRELATION_ID =
            "CORRELATION_ID";

    public static final String REMARK =
            "REMARK";

    public static final String CREATED_BY =
            "CREATED_BY";

    public static final int STACK_TRACE_LIMIT =
            10;

    public static final int QUEUE_SIZE =
            10000;

    public static final int BATCH_SIZE =
            50;
}
________________________________________
3.ExceptionTrackerAspect.java
package com.sbi.epay.exceptionTracker.aspect;

import com.sbi.epay.exceptionTracker.annotation.TrackException;
import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.service.ExceptionQueueService;
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
 * Description: Aspect class used to capture and process exceptions from annotated methods.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@Aspect
@Component
@RequiredArgsConstructor
public class ExceptionTrackerAspect {

    private final ExceptionQueueService queueService;

    @Value("${spring.application.name}")
    private String serviceName;

    // Intercepts methods annotated with @TrackException
    @Around("@annotation(trackException)")
    public Object trackException(
            ProceedingJoinPoint joinPoint,
            TrackException trackException)
            throws Throwable {

        try {

            // Executes actual business method
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
                                    signature.getDeclaringTypeName())
                            .methodName(
                                    signature.getMethod().getName())
                            .exceptionType(
                                    ex.getClass().getName())
                            .exceptionMessage(
                                    ex.getMessage())
                            .stackTrace(
                                    trackException.storeStackTrace()
                                            ? StackTraceUtil.getShortStackTrace(ex)
                                            : null)
                            .mdcMap(mdc)
                            .build();

            // Pushes exception details into queue
            queueService.add(dto);

            if (trackException.rethrow()) {

                throw ex;
            }

            return null;
        }
    }
}
________________________________________
4.QueueConfig.java
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
 * <p>
 * Version:1.0
 **/
@Configuration
public class QueueConfig {

    // Queue used for temporary exception storage
    @Bean
    public BlockingQueue<ExceptionLogDto>
    exceptionQueue() {

        // Queue Size : 10000 used to temporarily handle high exception traffic.
        return new LinkedBlockingQueue<>(
                ErrorConstant.QUEUE_SIZE);
    }
}
________________________________________
5.ExceptionLogDao.java
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
 * <p>
 * Version:1.0
 **/
@Component
@RequiredArgsConstructor
public class ExceptionLogDao {

    private final ExceptionLogRepository repository;

    public void saveAll(
            List<ExceptionLog> logs) {

        repository.saveAll(logs);
    }
}
________________________________________
6.ExceptionLogDto.java
package com.sbi.epay.exceptionTracker.dto;

import lombok.*;

import java.util.Map;

/**
 * Class Name: ExceptionLogDto
 * Description: DTO class used to transfer exception details between layers.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
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
________________________________________
7.ExceptionLog.java
package com.sbi.epay.exceptionTracker.entity;

import jakarta.persistence.*;

import lombok.*;

/**
 * Class Name: ExceptionLog
 * Description: Entity class mapped with EXCEPTION_LOG database table.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
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

    // Auto generated unique DB row id
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
________________________________________
8.ExceptionLogMapper.java
package com.sbi.epay.exceptionTracker.mapper;

import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.entity.ExceptionLog;

import org.mapstruct.Mapper;

/**
 * Class Name: ExceptionLogMapper
 * Description: Mapper interface used to convert ExceptionLogDto into ExceptionLog entity.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/

// MapStruct generates mapper implementation automatically
@Mapper(componentModel = "spring")
public interface ExceptionLogMapper {

    ExceptionLog toEntity(
            ExceptionLogDto dto);
}
________________________________________
9.ExceptionLogRepository.java
package com.sbi.epay.exceptionTracker.repository;

import com.sbi.epay.exceptionTracker.entity.ExceptionLog;

import org.springframework.data.jpa.repository.JpaRepository;

/**
 * Class Name: ExceptionLogRepository
 * Description: Repository interface used for ExceptionLog database operations.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
public interface ExceptionLogRepository
        extends JpaRepository<ExceptionLog, Long> {
}
________________________________________
10.ExceptionConsumerService.java
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

    private static final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(
                    ExceptionConsumerService.class);

    private final BlockingQueue<ExceptionLogDto>
            exceptionQueue;

    private final ExceptionLogDao dao;

    private final ExceptionLogMapper mapper;

    private final ObjectMapper objectMapper;

    // Starts background consumer thread after application startup
    @PostConstruct
    public void init() {

        Thread thread =
                new Thread(this::consume);

        thread.setDaemon(true);

        thread.start();
    }

    public void consume() {

        // Continuously listens for exception logs from queue
        while (true) {

            try {

                List<ExceptionLog> logs =
                        new ArrayList<>();

                // Waits until queue receives at least one log
                ExceptionLogDto dto =
                        exceptionQueue.take();

                prepare(dto);

                logs.add(
                        mapper.toEntity(dto));

                List<ExceptionLogDto> temp =
                        new ArrayList<>();

                // Batch Size : 50 used to reduce database calls and improve performance.
                // Fetches remaining logs in batch to reduce DB calls
                exceptionQueue.drainTo(
                        temp,
                        ErrorConstant.BATCH_SIZE);

                for (ExceptionLogDto data : temp) {

                    prepare(data);

                    logs.add(
                            mapper.toEntity(data));
                }

                // Saves logs into database using bulk insert
                dao.saveAll(logs);

            } catch (Exception ex) {

                logger.error(
                        "Error while saving exception logs",
                        ex);
            }
        }
    }

    // Maps MDC values into DTO before DB save
    private void prepare(
            ExceptionLogDto dto)
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

        // Converts MDC map into JSON string
        dto.setMdcJson(
                objectMapper.writeValueAsString(mdc));
    }
}
________________________________________
11.ExceptionQueueService.java
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

    private final BlockingQueue<ExceptionLogDto>
            exceptionQueue;

    public void add(
            ExceptionLogDto dto) {

        exceptionQueue.offer(dto);
    }
}
________________________________________
12.ExceptionUtil.java
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

    public static String getMessage(
            Throwable ex) {

        if (ex == null) {

            return null;
        }

        return ex.getMessage();
    }
}
________________________________________
13.MDCUtil.java
package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

import java.util.Map;

/**
 * Class Name: MDCUtil
 * Description: Utility class used to fetch MDC values ignoring key case sensitivity.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@UtilityClass
public class MDCUtil {

    // Fetches MDC value ignoring key case
    public static String getIgnoreCase(
            Map<String, String> map,
            String... keys) {

        if (map == null) {

            return null;
        }

        for (String key : keys) {

            for (Map.Entry<String, String> entry
                    : map.entrySet()) {

                if (entry.getKey()
                        .equalsIgnoreCase(key)) {

                    return entry.getValue();
                }
            }
        }

        return null;
    }
}
________________________________________
14.StackTraceUtil.java
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
    public static String getShortStackTrace(
            Throwable ex) {

        StringBuilder sb =
                new StringBuilder();

        sb.append(ex.getMessage())
                .append("
");

        StackTraceElement[] elements =
                ex.getStackTrace();

        // Stack Trace Limit : 10 used to avoid storing huge stack trace payload.
        int limit =
                Math.min(
                        elements.length,
                        ErrorConstant.STACK_TRACE_LIMIT);

        for (int i = 0; i < limit; i++) {

            sb.append(elements[i].toString())
                    .append("\n");
        }

        return sb.toString();
    }
}
________________________________________
15.EXCEPTION_LOG.sql
-- LIQUIBASE FORMATTED SQL
-- CHANGESET RohitG:1

CREATE TABLE EXCEPTION_LOG (
    ID                NUMBER GENERATED BY DEFAULT AS IDENTITY,
    SERVICE_NAME      VARCHAR2(50),
    CLASS_NAME        VARCHAR2(100),
    METHOD_NAME       VARCHAR2(100),
    EXCEPTION_TYPE    VARCHAR2(100),
    EXCEPTION_MESSAGE CLOB,
    STACK_TRACE       CLOB,
    MERCHANT_ID       VARCHAR2(20),
    ORDER_REF_NUMBER  VARCHAR2(50),
    ATRN_NUM          VARCHAR2(50),
    PAY_MODE          VARCHAR2(50),
    CORRELATION_ID    VARCHAR2(50),
    REMARK            VARCHAR2(500),
    MDC_JSON          CLOB,
    CREATED_BY        VARCHAR2(100),
    CREATED_DATE      NUMBER
);
