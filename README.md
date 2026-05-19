========================================
UTILITY CLASSES - FINAL CODE

========================================
ErrorConstant.java

package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

/**

* Class Name : ErrorConstant

* Description : Utility class used to store

* exception tracker constants.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
  @UtilityClass
  public class ErrorConstant {
  
  /**
  
  * MDC Keys
    */
    public static final String MID =
    "MID";
  
  public static final String CORRELATION_ID =
  "CORRELATION_ID";
  
  public static final String REMARK =
  "REMARK";
  
  public static final String CREATED_BY =
  "CREATED_BY";
  
  /**
  
  * Queue configuration
    */
    public static final int QUEUE_SIZE =
    10000;
  
  public static final int BATCH_SIZE =
  50;
  
  /**
  
  * Stack trace configuration
    */
    public static final int STACK_TRACE_LIMIT =
    10;
  
  /**
  
  * Default values
    */
    public static final String DEFAULT_EXCEPTION_MESSAGE =
    "Exception message not available";
  
  public static final String DEFAULT_MDC_VALUE =
  "MDC value not available";
  }

========================================
ExceptionUtil.java

package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

/**

* Class Name : ExceptionUtil

* Description : Utility class used to safely

* extract exception message.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
  @UtilityClass
  public class ExceptionUtil {
  
  /**
  
  * Returns safe exception message.
    */
    public static String getMessage(Throwable ex) {
    
    if (ex == null) {
    
     return ErrorConstant.DEFAULT_EXCEPTION_MESSAGE;
    
    }
    
    if (ex.getMessage() == null
    || ex.getMessage().trim().isEmpty()) {
    
     return ErrorConstant.DEFAULT_EXCEPTION_MESSAGE;
    
    }
    
    return ex.getMessage();
    }
    }

========================================
MDCUtil.java

package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

import java.util.Map;

/**

* Class Name : MDCUtil

* Description : Utility class used to safely

* fetch MDC values.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
  @UtilityClass
  public class MDCUtil {
  
  /**
  
  * Fetches MDC value ignoring key case.
    */
    public static String getIgnoreCase(
    Map<String, String> map,
    String... keys
    ) {
    
    if (map == null || map.isEmpty()) {
    
     return ErrorConstant.DEFAULT_MDC_VALUE;
    
    }
    
    for (String key : keys) {
    
     for (Map.Entry<String, String> entry
         : map.entrySet()) {

     if (entry.getKey()
             .equalsIgnoreCase(key)) {

         String value =
                 entry.getValue();

         if (value == null
                 || value.trim().isEmpty()) {

             return ErrorConstant.DEFAULT_MDC_VALUE;
         }

         return value;
     }
 }
    
    }
    
    return ErrorConstant.DEFAULT_MDC_VALUE;
    }
    }

========================================
StackTraceUtil.java

package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

/**

* Class Name : StackTraceUtil

* Description : Utility class used to generate

* formatted stack trace string from exception.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
  @UtilityClass
  public class StackTraceUtil {
  
  /**
  
  * Returns shortened stack trace.
    */
    public static String getShortStackTrace(
    Throwable ex
    ) {
    
    if (ex == null) {
    
     return null;
    
    }
    
    StringBuilder sb =
    new StringBuilder();
    
    sb.append(
    ExceptionUtil.getMessage(ex)
    ).append(" ");
    
    StackTraceElement[] elements =
    ex.getStackTrace();
    
    int limit =
    Math.min(
    elements.length,
    ErrorConstant.STACK_TRACE_LIMIT
    );
    
    for (int i = 0; i < limit; i++) {
    
     sb.append(
         elements[i].toString()
 ).append("\n");
    
    }
    
    return sb.toString();
    }
    }


========================================
FINAL ExceptionTrackerAspect.java

package com.sbi.epay.exceptionTracker.aspect;

import com.fasterxml.jackson.databind.ObjectMapper;

import com.sbi.epay.exceptionTracker.annotation.TrackException;
import com.sbi.epay.exceptionTracker.entity.ExceptionLog;
import com.sbi.epay.exceptionTracker.service.ExceptionQueueService;
import com.sbi.epay.exceptionTracker.util.ErrorConstant;
import com.sbi.epay.exceptionTracker.util.ExceptionUtil;
import com.sbi.epay.exceptionTracker.util.MDCUtil;
import com.sbi.epay.exceptionTracker.util.StackTraceUtil;

import lombok.RequiredArgsConstructor;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;

import org.slf4j.MDC;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.util.HashMap;
import java.util.Map;

/**

* Class Name : ExceptionTrackerAspect

* Description : Aspect class used to capture

* and prepare exception entity before queue processing.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
  @Aspect
  @Component
  @RequiredArgsConstructor
  public class ExceptionTrackerAspect {
  
  private final ExceptionQueueService queueService;
  
  private final ObjectMapper objectMapper;
  
  @Value("${spring.application.name}")
  private String serviceName;
  
  /**
  
  * Intercepts methods annotated with @TrackException.
    */
    @Around("@annotation(trackException)")
    public Object trackException(
    ProceedingJoinPoint joinPoint,
    TrackException trackException
    ) throws Throwable {
    
    try {
    
     /**
  * Executes actual business method.
  */
 return joinPoint.proceed();
    
    } catch (Exception ex) {
    
     MethodSignature signature =
         (MethodSignature) joinPoint.getSignature();

 Map<String, String> mdc =
         MDC.getCopyOfContextMap();

 if (mdc == null) {

     mdc = new HashMap<>();
 }

 /**
  * Entity preparation moved into AOP layer.
  */
 ExceptionLog entity =
         ExceptionLog.builder()
                 .serviceName(serviceName)
                 .className(
                         signature
                                 .getDeclaringTypeName()
                 )
                 .methodName(
                         signature
                                 .getMethod()
                                 .getName()
                 )
                 .exceptionType(
                         ex.getClass().getName()
                 )
                 .exceptionMessage(
                         ExceptionUtil.getMessage(ex)
                 )
                 .stackTrace(
                         trackException.storeStackTrace()
                                 ? StackTraceUtil
                                 .getShortStackTrace(ex)
                                 : null
                 )
                 .merchantId(
                         MDCUtil.getIgnoreCase(
                                 mdc,
                                 ErrorConstant.MID
                         )
                 )
                 .correlationId(
                         MDCUtil.getIgnoreCase(
                                 mdc,
                                 ErrorConstant.CORRELATION_ID
                         )
                 )
                 .remark(
                         MDCUtil.getIgnoreCase(
                                 mdc,
                                 ErrorConstant.REMARK
                         )
                 )
                 .createdBy(
                         MDCUtil.getIgnoreCase(
                                 mdc,
                                 ErrorConstant.CREATED_BY
                         )
                 )
                 .createdDate(
                         System.currentTimeMillis()
                 )
                 .mdcJson(
                         objectMapper.writeValueAsString(mdc)
                 )
                 .build();

 /**
  * Pushes prepared entity into queue.
  */
 queueService.add(entity);

 if (trackException.rethrow()) {

     throw ex;
 }

 return null;
    
    }
    }
    }

========================================
FINAL ExceptionBufferedRepository.java

package com.sbi.epay.exceptionTracker.repository;

import com.sbi.epay.exceptionTracker.entity.ExceptionLog;
import com.sbi.epay.exceptionTracker.util.ErrorConstant;

import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;

import jakarta.annotation.PostConstruct;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.core.task.TaskExecutor;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

/**

* Class Name : ExceptionBufferedRepository

* Description : Repository class used for

* asynchronous batch save processing.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
  @Component
  public class ExceptionBufferedRepository {
  
  private static final LoggerUtility logger =
  LoggerFactoryUtility.getLogger(
  ExceptionBufferedRepository.class
  );
  
  private final ExceptionLogRepository repository;
  
  private final TaskExecutor taskExecutor;
  
  /**
  
  * Queue used for storing prepared entities.
    */
    private final BlockingQueue<ExceptionLog> queue =
    new LinkedBlockingQueue<>(
    ErrorConstant.QUEUE_SIZE
    );
  
  public ExceptionBufferedRepository(
  ExceptionLogRepository repository,
  
       @Qualifier("exceptionTaskExecutor")
     TaskExecutor taskExecutor
  
  ) {
  
   this.repository = repository;
 this.taskExecutor = taskExecutor;
  
  }
  
  /**
  
  * Starts async consumer thread.
    */
    @PostConstruct
    public void init() {
    
    taskExecutor.execute(this::saveAndFlush);
    }
  
  /**
  
  * Adds entity into queue.
    */
    public void buffer(ExceptionLog entity) {
    
    if (!queue.offer(entity)) {
    
     logger.error(
         "Error while buffering exception log, queue full"
 );
    
    }
    }
  
  /**
  
  * Consumes queue data and performs direct bulk save.
    */
    private void saveAndFlush() {
    
    while (true) {
    
     try {

     List<ExceptionLog> batch =
             new ArrayList<>();

     ExceptionLog first =
             queue.poll(
                     1,
                     TimeUnit.SECONDS
             );

     if (first != null) {

         batch.add(first);

         queue.drainTo(
                 batch,
                 ErrorConstant.BATCH_SIZE
         );

         /**
          * Direct database bulk save.
          */
         repository.saveAll(batch);

         logger.info(
                 "Exception log batch saved successfully, batch size : {}",
                 batch.size()
         );
     }

 } catch (Exception ex) {

     logger.error(
             "Error while saving exception log batch",
             ex
     );
 }
    
    }
    }
    }

========================================
FINAL ExceptionQueueService.java

package com.sbi.epay.exceptionTracker.service;

import com.sbi.epay.exceptionTracker.entity.ExceptionLog;
import com.sbi.epay.exceptionTracker.repository.ExceptionBufferedRepository;

import lombok.RequiredArgsConstructor;

import org.springframework.stereotype.Service;

/**

* Class Name : ExceptionQueueService

* Description : Service class used to push

* exception entities into buffer queue.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
  @Service
  @RequiredArgsConstructor
  public class ExceptionQueueService {
  
  private final ExceptionBufferedRepository repository;
  
  /**
  
  * Pushes entity into queue buffer.
    */
    public void add(ExceptionLog entity) {
    
    repository.buffer(entity);
    }
    }

========================================
FINAL ExceptionLog.java

package com.sbi.epay.exceptionTracker.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Lob;
import jakarta.persistence.Table;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

/**

* Class Name : ExceptionLog

* Description : Entity class mapped with

* EXCEPTION_LOG table.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
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
  
  private String correlationId;
  
  private String remark;
  
  @Lob
  private String mdcJson;
  
  private String createdBy;
  
  private Long createdDate;
  }


  


    
