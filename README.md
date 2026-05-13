package com.sbi.epay.exceptionTracker.annotation;

import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface TrackException {

    boolean storeStackTrace() default true;

    boolean rethrow() default true;
}

========================================================

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
            MethodSignature signature = (MethodSignature) joinPoint.getSignature();
            Map<String, String> mdc = MDC.getCopyOfContextMap();
            ExceptionLogDto dto = ExceptionLogDto.builder()
                            .serviceName(serviceName)
                            .className(signature.getDeclaringTypeName())
                            .methodName(signature.getMethod().getName())
                            .exceptionType(ex.getClass().getName())
                            .exceptionMessage(ex.getMessage())
                            .stackTrace(trackException.storeStackTrace() ? StackTraceUtil.getShortStackTrace(ex) : null)
                            .mdcMap(mdc)
                            .build();
            queueService.add(dto);
            if (trackException.rethrow()) {
                throw ex;
            }
            return null;
        }
    }
}
========================================================
package com.sbi.epay.exceptionTracker.config;

import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import org.springframework.context.annotation.*;
import java.util.concurrent.*;

@Configuration
public class QueueConfig {

    @Bean
    public BlockingQueue<ExceptionLogDto> exceptionQueue() {

        return new LinkedBlockingQueue<>(10000);
    }
}

========================================================
package com.sbi.epay.exceptionTracker.dao;

import com.sbi.epay.exceptionTracker.entity.ExceptionLog;
import com.sbi.epay.exceptionTracker.repository.ExceptionLogRepository;

import lombok.RequiredArgsConstructor;

import org.springframework.stereotype.Component;

import java.util.List;

@Component
@RequiredArgsConstructor
public class ExceptionLogDao {

    private final ExceptionLogRepository repository;

    public void saveAll(List<ExceptionLog> logs) {
        repository.saveAll(logs);
    }
}

========================================================

package com.sbi.epay.exceptionTracker.dto;

import lombok.*;
import java.util.Map;

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

========================================================
package com.sbi.epay.exceptionTracker.entity;

import jakarta.persistence.*;
import lombok.*;



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
    private String remark;
    @Lob
    private String mdcJson;
    private String createdBy;
    private Long createdDate;
}


========================================================

package com.sbi.epay.exceptionTracker.mapper;

import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.entity.ExceptionLog;

import org.springframework.stereotype.Component;

@Component
public class ExceptionLogMapper {

    public ExceptionLog toEntity(
            ExceptionLogDto dto) {

        return ExceptionLog.builder()
                .serviceName(dto.getServiceName())
                .className(dto.getClassName())
                .methodName(dto.getMethodName())
                .exceptionType(dto.getExceptionType())
                .exceptionMessage(dto.getExceptionMessage())
                .stackTrace(dto.getStackTrace())
                .merchantId(dto.getMerchantId())
                .orderRefNumber(dto.getOrderRefNumber())
                .atrnNum(dto.getAtrnNum())
                .payMode(dto.getPayMode())
                .remark(dto.getRemark())
                .mdcJson(dto.getMdcJson())
                .createdBy(dto.getCreatedBy())
                .createdDate(dto.getCreatedDate())
                .build();
    }
}

========================================================

package com.sbi.epay.exceptionTracker.repository;

import com.sbi.epay.exceptionTracker.entity.ExceptionLog;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ExceptionLogRepository extends JpaRepository<ExceptionLog, String> {
}
========================================================
package com.sbi.epay.exceptionTracker.service;

import com.fasterxml.jackson.databind.ObjectMapper;

import com.sbi.epay.exceptionTracker.dao.ExceptionLogDao;
import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.entity.ExceptionLog;
import com.sbi.epay.exceptionTracker.mapper.ExceptionLogMapper;
import com.sbi.epay.exceptionTracker.util.MDCUtil;
import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;


import jakarta.annotation.PostConstruct;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.BlockingQueue;

@Service
@RequiredArgsConstructor
@Slf4j
public class ExceptionConsumerService {

    private final BlockingQueue<ExceptionLogDto> exceptionQueue;
    private static final LoggerUtility logger = LoggerFactoryUtility.getLogger(ExceptionConsumerService.class);

    private final ExceptionLogDao dao;
    private final ExceptionLogMapper mapper;
    private final ObjectMapper objectMapper;

    @PostConstruct
    public void init() {
        Thread thread = new Thread(this::consume);
        thread.setDaemon(true);
        thread.start();
    }

    public void consume() {

        while (true) {
            try {
                List<ExceptionLog> logs = new ArrayList<>();
                ExceptionLogDto dto = exceptionQueue.take();
                prepare(dto);
                logs.add(mapper.toEntity(dto));
                List<ExceptionLogDto> temp = new ArrayList<>();
                exceptionQueue.drainTo(temp, 50);
                for (ExceptionLogDto data : temp) {
                    prepare(data);
                    logs.add(mapper.toEntity(data));
                }
                dao.saveAll(logs);
            } catch (Exception ex) {
                logger.error("Error while saving exception logs", ex);
            }
        }
    }

    private void prepare(ExceptionLogDto dto)
            throws Exception {

        Map<String, String> mdc = dto.getMdcMap();
        dto.setMerchantId(MDCUtil.getIgnoreCase(mdc, "MID"));
        dto.setOrderRefNumber(MDCUtil.getIgnoreCase(mdc, "ORDER_REF"));
        dto.setAtrnNum(MDCUtil.getIgnoreCase(mdc, "ATRN"));
        dto.setPayMode(MDCUtil.getIgnoreCase(mdc, "PAYMODE"));
        dto.setCorrelationId(MDCUtil.getIgnoreCase(mdc, "CORRELATION_ID"));
        dto.setRemark(MDCUtil.getIgnoreCase(mdc, "REMARK"));
        dto.setCreatedBy("SYSTEM");
        dto.setCreatedDate(System.currentTimeMillis());
        dto.setMdcJson(objectMapper.writeValueAsString(mdc));
    }
}

========================================================
package com.sbi.epay.exceptionTracker.service;

import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;

import lombok.RequiredArgsConstructor;

import org.springframework.stereotype.Service;

import java.util.concurrent.BlockingQueue;

@Service
@RequiredArgsConstructor
public class ExceptionQueueService {

    private final BlockingQueue<ExceptionLogDto> exceptionQueue;
    public void add(ExceptionLogDto dto) {
        exceptionQueue.offer(dto);
    }
}

========================================================
package com.sbi.epay.exceptionTracker.util;

public class ExceptionUtil {

    public static String getMessage(Throwable ex) {
        if (ex == null) {
            return null;
        }
        return ex.getMessage();
    }
}
========================================================

package com.sbi.epay.exceptionTracker.util;

import java.util.Map;

public class MDCUtil {

    public static String getIgnoreCase(
            Map<String, String> map,
            String... keys) {

        if (map == null) {
            return null;
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
        return null;
    }
}
========================================================
package com.sbi.epay.exceptionTracker.util;

public class StackTraceUtil {

    public static String getShortStackTrace(Throwable ex) {
        StringBuilder sb = new StringBuilder();
        sb.append(ex.getMessage()).append("\n");
        StackTraceElement[] elements = ex.getStackTrace();
        int limit = Math.min(elements.length, 10);
        for (int i = 0; i < limit; i++) {
            sb.append(elements[i].toString())
                    .append("\n");
        }
        return sb.toString();
    }
}

========================================================


plugins {
    id 'java'
    id 'org.springframework.boot' version "${spring_boot}"
    id 'io.spring.dependency-management' version "${dependency_plugin}"
}

group = 'com.epay.utility'
version = "${version}"

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}
configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
    flatDir {
        dirs "libs"
    }
    maven {
        url "https://gitlab.epay.sbi/api/v4/projects/16/packages/maven"
        credentials(PasswordCredentials) {
            username = project.findProperty("gitlab.username")?: System.getenv("CI_USERNAME")
            password = project.findProperty("gitlab.token")?: System.getenv("CI_JOB_TOKEN")
        }
        authentication {
            basic(BasicAuthentication)
        }
    }
}

dependencies {

    implementation 'org.springframework.boot:spring-boot-starter'
    implementation 'org.springframework.boot:spring-boot-starter-aop'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

    implementation "org.apache.commons:commons-lang3:${commons_lang3}"
    implementation 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    implementation "org.mapstruct:mapstruct:${mapstruct}"
    annotationProcessor "org.mapstruct:mapstruct-processor:${mapstruct}"

    implementation "org.projectlombok:lombok-mapstruct-binding:${lombok_mapstruct}"
    implementation "com.sbi.epay:logging-service:${sbi_logging}"
    implementation "com.oracle.database.jdbc:ojdbc11:${oracle_driver}"
    implementation "com.fasterxml.jackson.core:jackson-databind:${jacksonVersion}"

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

configurations {
    all*.exclude module: 'spring-boot-starter-logging'
    all*.exclude module: 'slf4j-simple'
}

tasks.named('test') {
    useJUnitPlatform()
}

bootJar {
    enabled = false
}

jar {
    archiveClassifier = ''
    enabled = true
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
}

========================================================
version=1.0.0
spring_boot=3.5.8
dependency_plugin=1.1.7
oracle_driver=23.5.0.24.07
mapstruct=1.5.1.Final
lombok_mapstruct=0.2.0
commons_lang3=3.18.0
jacksonVersion=2.17.0
sbi_logging=1.0.0

========================================================
-- LIQUIBASE FORMATTED SQL
-- CHANGESET RohitG:1

CREATE TABLE EXCEPTION_LOG (
    ID NUMBER GENERATED BY DEFAULT AS IDENTITY,
    SERVICE_NAME                 VARCHAR2(100),
    CLASS_NAME                   VARCHAR2(500),
    METHOD_NAME                  VARCHAR2(500),
    EXCEPTION_TYPE               VARCHAR2(500),
    EXCEPTION_MESSAGE                     CLOB,
    STACK_TRACE                           CLOB,
    MERCHANT_ID                   VARCHAR2(20),
    ORDER_REF_NUMBER              VARCHAR2(50),
    ATRN_NUM                      VARCHAR2(50),
    PAY_MODE                      VARCHAR2(50),
    CORRELATION_ID                VARCHAR2(50),
    REMARK                       VARCHAR2(200),
    MDC_JSON                              CLOB,
    CREATED_BY              VARCHAR2(100 BYTE),
    CREATED_DATE                        NUMBER
);

========================================================
/**
 * Class Name: StackTraceUtil
 * Description: Utility class used to generate formatted stack trace string from exception.
 * <p>
 * Author: V1024113(Rohit Gardi)
 * <p>
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * <p>
 * Version:1.0
 **/
 
 he assa sare class madhe add karaycha ahe tyclass cha name and Description

========================================================
