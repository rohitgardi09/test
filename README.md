Exception Tracker Final Complete Structure

01. com.epay.operations.annotation
    └── TrackException.java

02. com.epay.operations.aspect
    └── ExceptionTrackerAspect.java

03. com.epay.operations.config
    └── QueueConfig.java

04. com.epay.operations.dao
    └── ExceptionLogDao.java

05. com.epay.operations.dto
    ├── BaseExceptionDto.java
    └── PaymentExceptionDto.java

06. com.epay.operations.entity
    ├── BaseExceptionEntity.java
    └── PaymentExceptionLog.java

07. com.epay.operations.mapper
    └── ExceptionLogMapper.java

08. com.epay.operations.repository
    └── ExceptionLogRepository.java

09. com.epay.operations.scheduler
    └── ExceptionCleanupScheduler.java

10. com.epay.operations.service
    ├── ExceptionConsumerService.java
    └── ExceptionQueueService.java

11. com.epay.operations.util
    ├── ErrorConstant.java
    ├── ExceptionUtil.java
    ├── MDCUtil.java
    └── StackTraceUtil.java

12. com.epay.operations.controller
    └── TestController.java

---

01. TrackException.java

package com.epay.operations.annotation;

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

---

02. BaseExceptionDto.java

package com.epay.operations.dto;

import lombok.*;
import lombok.experimental.SuperBuilder;

import java.util.Map;

/**
 * Class Name: BaseExceptionDto
 * Description: Generic base DTO used for common exception logging fields.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@Getter
@Setter
@SuperBuilder
@NoArgsConstructor
@AllArgsConstructor
public class BaseExceptionDto {

    private String serviceName;

    private String className;

    private String methodName;

    private String exceptionType;

    private String exceptionMessage;

    private String stackTrace;

    private String correlationId;

    private String remark;

    private String createdBy;

    private Long createdDate;

    private String mdcJson;

    private Map<String, String> mdcMap;
}

---

03. PaymentExceptionDto.java

package com.epay.operations.dto;

import lombok.*;
import lombok.experimental.SuperBuilder;

/**
 * Class Name: PaymentExceptionDto
 * Description: Service specific DTO used for payment related exception fields.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@Getter
@Setter
@SuperBuilder
@NoArgsConstructor
@EqualsAndHashCode(callSuper = true)
public class PaymentExceptionDto
        extends BaseExceptionDto {

    private String merchantId;

    private String orderRefNumber;

    private String atrnNum;

    private String payMode;
}

---

04. BaseExceptionEntity.java

package com.epay.operations.entity;

import jakarta.persistence.Column;
import jakarta.persistence.MappedSuperclass;

import lombok.Getter;
import lombok.Setter;

/**
 * Class Name: BaseExceptionEntity
 * Description: Generic base entity used for common exception fields.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@Getter
@Setter
@MappedSuperclass
public class BaseExceptionEntity {

    @Column(name = "SERVICE_NAME")
    private String serviceName;

    @Column(name = "CLASS_NAME")
    private String className;

    @Column(name = "METHOD_NAME")
    private String methodName;

    @Column(name = "EXCEPTION_TYPE")
    private String exceptionType;

    @Column(name = "EXCEPTION_MESSAGE")
    private String exceptionMessage;

    @Column(name = "STACK_TRACE")
    private String stackTrace;

    @Column(name = "CORRELATION_ID")
    private String correlationId;

    @Column(name = "REMARK")
    private String remark;

    @Column(name = "CREATED_BY")
    private String createdBy;

    @Column(name = "CREATED_DATE")
    private Long createdDate;

    @Column(name = "MDC_JSON")
    private String mdcJson;
}

---

05. PaymentExceptionLog.java

package com.epay.operations.entity;

import jakarta.persistence.*;

import lombok.Getter;
import lombok.Setter;

/**
 * Class Name: PaymentExceptionLog
 * Description: Service specific entity used for payment exception logs.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@Getter
@Setter
@Entity
@Table(name = "EXCEPTION_LOG")
public class PaymentExceptionLog
        extends BaseExceptionEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "MERCHANT_ID")
    private String merchantId;

    @Column(name = "ORDER_REF_NUMBER")
    private String orderRefNumber;

    @Column(name = "ATRN_NUM")
    private String atrnNum;

    @Column(name = "PAY_MODE")
    private String payMode;
}

Remaining Final Files

06. ExceptionTrackerAspect.java

package com.epay.operations.aspect;

import com.epay.operations.annotation.TrackException;
import com.epay.operations.dto.PaymentExceptionDto;
import com.epay.operations.service.ExceptionQueueService;
import com.epay.operations.util.ExceptionUtil;
import com.epay.operations.util.StackTraceUtil;

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
 * Description: Aspect class used to capture and push exception logs into queue.
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

            PaymentExceptionDto dto =
                    PaymentExceptionDto.builder()
                            .serviceName(serviceName)
                            .className(
                                    signature.getDeclaringTypeName())
                            .methodName(
                                    signature.getMethod()
                                            .getName())
                            .exceptionType(
                                    ex.getClass().getName())
                            .exceptionMessage(
                                    ExceptionUtil.getMessage(ex))
                            .stackTrace(
                                    trackException.storeStackTrace()
                                            ? StackTraceUtil
                                            .getShortStackTrace(ex)
                                            : "")
                            .correlationId(
                                    MDC.get("CORRELATION_ID"))
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

---

07. QueueConfig.java

package com.epay.operations.config;

import com.epay.operations.dto.BaseExceptionDto;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

/**
 * Class Name: QueueConfig
 * Description: Configuration class used to create exception queue bean.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@Configuration
public class QueueConfig {

    @Bean
    public BlockingQueue<BaseExceptionDto>
    exceptionQueue() {

        return new LinkedBlockingQueue<>(10000);
    }
}

---

08. ExceptionLogDao.java

package com.epay.operations.dao;

import com.epay.operations.entity.PaymentExceptionLog;
import com.epay.operations.repository.ExceptionLogRepository;

import lombok.RequiredArgsConstructor;

import org.springframework.stereotype.Component;

/**
 * Class Name: ExceptionLogDao
 * Description: DAO class used for exception log DB operations.
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

    public void save(
            PaymentExceptionLog entity) {

        repository.save(entity);
    }
}

---

09. ExceptionLogMapper.java

package com.epay.operations.mapper;

import com.epay.operations.dto.PaymentExceptionDto;
import com.epay.operations.entity.PaymentExceptionLog;

import org.mapstruct.Mapper;
import org.mapstruct.Mapping;

/**
 * Class Name: ExceptionLogMapper
 * Description: Mapper interface used to convert DTO into entity.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@Mapper(componentModel = "spring")
public interface ExceptionLogMapper {

    @Mapping(
            target = "id",
            ignore = true)
    PaymentExceptionLog toEntity(
            PaymentExceptionDto dto);
}

---

10. ExceptionLogRepository.java

package com.epay.operations.repository;

import com.epay.operations.entity.PaymentExceptionLog;

import jakarta.transaction.Transactional;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;

/**
 * Class Name: ExceptionLogRepository
 * Description: Repository interface used for exception log DB operations.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
public interface ExceptionLogRepository
        extends JpaRepository<
        PaymentExceptionLog,
        Long> {

    @Modifying
    @Transactional
    @Query(
            value = "DELETE FROM EXCEPTION_LOG "
                    + "WHERE CREATED_DATE < :time",
            nativeQuery = true)
    void deleteOldLogs(
            Long time);
}

---

11. ExceptionCleanupScheduler.java

package com.epay.operations.scheduler;

import com.epay.operations.repository.ExceptionLogRepository;

import lombok.RequiredArgsConstructor;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

/**
 * Class Name: ExceptionCleanupScheduler
 * Description: Scheduler class used to delete old exception logs.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@Component
@RequiredArgsConstructor
public class ExceptionCleanupScheduler {

    private final ExceptionLogRepository repository;

    @Scheduled(cron = "0 0 2 * * ?")
    public void deleteOldLogs() {

        long sevenDaysOldTime =
                System.currentTimeMillis()
                        - (7L * 24 * 60 * 60 * 1000);

        repository.deleteOldLogs(
                sevenDaysOldTime);
    }
}



Final Remaining Files - Part 3

12. ExceptionConsumerService.java

package com.epay.operations.service;

import com.fasterxml.jackson.databind.ObjectMapper;

import com.epay.operations.dao.ExceptionLogDao;
import com.epay.operations.dto.BaseExceptionDto;
import com.epay.operations.dto.PaymentExceptionDto;
import com.epay.operations.entity.PaymentExceptionLog;
import com.epay.operations.mapper.ExceptionLogMapper;
import com.epay.operations.util.MDCUtil;

import jakarta.annotation.PostConstruct;

import lombok.RequiredArgsConstructor;

import org.springframework.stereotype.Service;

import java.util.Map;
import java.util.concurrent.BlockingQueue;

/**
 * Class Name: ExceptionConsumerService
 * Description: Service class used to consume queue and save exception logs.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@Service
@RequiredArgsConstructor
public class ExceptionConsumerService {

    private final BlockingQueue<BaseExceptionDto>
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

                BaseExceptionDto dto =
                        exceptionQueue.take();

                if (dto instanceof
                        PaymentExceptionDto paymentDto) {

                    prepare(paymentDto);

                    PaymentExceptionLog entity =
                            mapper.toEntity(paymentDto);

                    dao.save(entity);
                }

            } catch (Exception ex) {

                ex.printStackTrace();
            }
        }
    }

    private void prepare(
            PaymentExceptionDto dto)
            throws Exception {

        Map<String, String> mdc =
                dto.getMdcMap();

        dto.setMerchantId(
                MDCUtil.getIgnoreCase(
                        mdc,
                        "MID"));

        dto.setOrderRefNumber(
                MDCUtil.getIgnoreCase(
                        mdc,
                        "ORDER_REF"));

        dto.setAtrnNum(
                MDCUtil.getIgnoreCase(
                        mdc,
                        "ATRN"));

        dto.setPayMode(
                MDCUtil.getIgnoreCase(
                        mdc,
                        "PAYMODE"));

        dto.setCreatedBy(
                MDCUtil.getIgnoreCase(
                        mdc,
                        "CREATED_BY"));

        dto.setCreatedDate(
                System.currentTimeMillis());

        dto.setMdcJson(
                objectMapper.writeValueAsString(mdc));
    }
}

---

13. ExceptionQueueService.java

package com.epay.operations.service;

import com.epay.operations.dto.BaseExceptionDto;

import lombok.RequiredArgsConstructor;

import org.springframework.stereotype.Service;

import java.util.concurrent.BlockingQueue;

/**
 * Class Name: ExceptionQueueService
 * Description: Service class used to push exception details into queue.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@Service
@RequiredArgsConstructor
public class ExceptionQueueService {

    private final BlockingQueue<BaseExceptionDto>
            exceptionQueue;

    public void add(
            BaseExceptionDto dto) {

        exceptionQueue.offer(dto);
    }
}

---

14. ErrorConstant.java

package com.epay.operations.util;

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

    public static final String MID =
            "MID";

    public static final String ORDER_REF =
            "ORDER_REF";

    public static final String ATRN =
            "ATRN";

    public static final String PAYMODE =
            "PAYMODE";

    public static final String CREATED_BY =
            "CREATED_BY";

    public static final String CORRELATION_ID =
            "CORRELATION_ID";

    public static final int STACK_TRACE_LIMIT =
            10;
}

---

15. ExceptionUtil.java

package com.epay.operations.util;

import lombok.experimental.UtilityClass;

/**
 * Class Name: ExceptionUtil
 * Description: Utility class used to safely extract exception message.
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

            return "";
        }

        if (ex.getMessage() == null) {

            return ex.getClass()
                    .getSimpleName();
        }

        return ex.getMessage();
    }
}
Final Remaining Files - Last Part

16. MDCUtil.java

package com.epay.operations.util;

import lombok.experimental.UtilityClass;

import java.util.Map;

/**
 * Class Name: MDCUtil
 * Description: Utility class used to fetch MDC values ignoring case sensitivity.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@UtilityClass
public class MDCUtil {

    public static String getIgnoreCase(
            Map<String, String> map,
            String... keys) {

        if (map == null) {

            return "";
        }

        for (String key : keys) {

            for (Map.Entry<String, String>
                    entry : map.entrySet()) {

                if (entry.getKey()
                        .equalsIgnoreCase(key)) {

                    return entry.getValue();
                }
            }
        }

        return "";
    }
}

---

17. StackTraceUtil.java

package com.epay.operations.util;

import lombok.experimental.UtilityClass;

/**
 * Class Name: StackTraceUtil
 * Description: Utility class used to generate formatted stack trace string.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@UtilityClass
public class StackTraceUtil {

    public static String getShortStackTrace(
            Throwable ex) {

        StringBuilder sb =
                new StringBuilder();

        sb.append(ex.getMessage())
                .append("\n");

        StackTraceElement[] elements =
                ex.getStackTrace();

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

---

18. TestController.java

package com.epay.operations.controller;

import com.epay.operations.annotation.TrackException;

import org.slf4j.MDC;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

/**
 * Class Name: TestController
 * Description: Test controller used to validate exception tracking flow.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
@RestController
public class TestController {

    @GetMapping("/test-runtime-exception")
    @TrackException
    public String runtimeException() {

        setMdcValues();

        throw new RuntimeException(
                "Runtime exception generated");
    }

    private void setMdcValues() {

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
                "CREATED_BY",
                System.getProperty(
                        "user.name"));
    }
}

---

19. EXCEPTION_LOG.sql

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
);



