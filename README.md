@Mapper(componentModel = "spring")
public interface ExceptionLogMapper {

    @Mapping(target = "id", ignore = true)

    @Mapping(
        target = "mdcJson",
        expression = "java(dto.getMdcMap() != null && !dto.getMdcMap().isEmpty() ? dto.getMdcJson() : null)"
    )

    ExceptionLog toEntity(ExceptionLogDto dto);
}



?????#######
@RestController
@RequestMapping("/test")
@RequiredArgsConstructor
public class TestController {

    @GetMapping("/exception")
    @TrackException(storeStackTrace = true, rethrow = false)
    public String testException() {

        // MDC values
        MDC.put("MID", "MID123456");
        MDC.put("CORRELATION_ID", "CORR-123456789");
        MDC.put("REMARK", "Testing Exception Tracker");
        MDC.put("CREATED_BY", "Rohit");
        MDC.put("ORDER_REF", "ORDER123");
        MDC.put("ATRN", "ATRN123");
        MDC.put("PAYMODE", "UPI");

        // Test Exception
        String value = null;
        value.length();

        return "SUCCESS";
    }
}

###




1. ExceptionQueueService.java

package com.sbi.epay.exceptionTracker.service;

import com.fasterxml.jackson.databind.ObjectMapper;

import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.repository.ExceptionBufferedRepository;
import com.sbi.epay.exceptionTracker.util.ErrorConstant;
import com.sbi.epay.exceptionTracker.util.MDCUtil;

import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;

import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;

/**
 * Class Name : ExceptionQueueService
 * Description : Service class used to prepare and buffer exception logs.
 * Author : V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * Version:1.0
 */
@Service
public class ExceptionQueueService {

    private static final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(
                    ExceptionQueueService.class
            );

    private final ExceptionBufferedRepository
            exceptionBufferedRepository;

    private final ObjectMapper objectMapper;

    public ExceptionQueueService(
            ExceptionBufferedRepository
                    exceptionBufferedRepository,
            ObjectMapper objectMapper
    ) {

        this.exceptionBufferedRepository =
                exceptionBufferedRepository;

        this.objectMapper = objectMapper;
    }

    /**
     * Prepares DTO and pushes into buffer.
     */
    public void add(ExceptionLogDto dto) {

        try {

            Map<String, String> mdc =
                    dto.getMdcMap();

            if (mdc == null) {
                mdc = new HashMap<>();
            }

            dto.setMerchantId(
                    MDCUtil.getIgnoreCase(
                            mdc,
                            ErrorConstant.MID
                    )
            );

            dto.setCorrelationId(
                    MDCUtil.getIgnoreCase(
                            mdc,
                            ErrorConstant.CORRELATION_ID
                    )
            );

            dto.setRemark(
                    MDCUtil.getIgnoreCase(
                            mdc,
                            ErrorConstant.REMARK
                    )
            );

            dto.setCreatedBy(
                    MDCUtil.getIgnoreCase(
                            mdc,
                            ErrorConstant.CREATED_BY
                    )
            );

            dto.setCreatedDate(
                    System.currentTimeMillis()
            );

            dto.setMdcJson(
                    objectMapper.writeValueAsString(mdc)
            );

            exceptionBufferedRepository.buffer(dto);

        } catch (Exception ex) {

            logger.error(
                    "Error while preparing exception log",
                    ex
            );
        }
    }
}

====================================================

2. ExceptionBufferedRepository.java

package com.sbi.epay.exceptionTracker.repository;

import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.entity.ExceptionLog;
import com.sbi.epay.exceptionTracker.mapper.ExceptionLogMapper;
import com.sbi.epay.exceptionTracker.util.ErrorConstant;

import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;

import jakarta.annotation.PostConstruct;

import org.apache.commons.lang3.ObjectUtils;

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
 * Description : Buffered repository used for asynchronous
 * batch processing of exception logs.
 * Author : V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * Version:1.0
 */
@Component
public class ExceptionBufferedRepository {

    private static final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(
                    ExceptionBufferedRepository.class
            );

    private final ExceptionLogRepository repository;

    private final ExceptionLogMapper mapper;

    private final TaskExecutor taskExecutor;

    private final BlockingQueue<ExceptionLogDto> queue =
            new LinkedBlockingQueue<>(
                    ErrorConstant.QUEUE_SIZE
            );

    public ExceptionBufferedRepository(
            ExceptionLogRepository repository,
            ExceptionLogMapper mapper,
            @Qualifier("exceptionTaskExecutor")
            TaskExecutor taskExecutor
    ) {

        this.repository = repository;
        this.mapper = mapper;
        this.taskExecutor = taskExecutor;
    }

    /**
     * Starts async batch consumer thread.
     */
    @PostConstruct
    public void init() {

        taskExecutor.execute(this::saveAndFlush);
    }

    /**
     * Adds exception log into queue.
     */
    public void buffer(ExceptionLogDto dto) {

        if (!queue.offer(dto)) {

            logger.error(
                    "Error while buffering exception log, queue full"
            );
        }
    }

    /**
     * Consumes queue data and performs bulk save.
     */
    private void saveAndFlush() {

        while (true) {

            try {

                List<ExceptionLog> batch =
                        new ArrayList<>();

                ExceptionLogDto first =
                        queue.poll(
                                1,
                                TimeUnit.SECONDS
                        );

                if (ObjectUtils.isNotEmpty(first)) {

                    try {

                        batch.add(
                                mapper.toEntity(first)
                        );

                    } catch (Exception ex) {

                        logger.error(
                                "Error while mapping first exception log",
                                ex
                        );
                    }

                    List<ExceptionLogDto> temp =
                            new ArrayList<>();

                    queue.drainTo(
                            temp,
                            ErrorConstant.BATCH_SIZE
                    );

                    for (ExceptionLogDto dto : temp) {

                        try {

                            batch.add(
                                    mapper.toEntity(dto)
                            );

                        } catch (Exception ex) {

                            logger.error(
                                    "Error while mapping exception log",
                                    ex
                            );
                        }
                    }

                    if (!batch.isEmpty()) {

                        repository.saveAll(batch);

                        logger.info(
                                "Exception log batch saved successfully, batch size : {}",
                                batch.size()
                        );
                    }
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

====================================================

3. ErrorConstant.java

package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

/**
 * Class Name : ErrorConstant
 * Description : Utility class used to store exception tracker constants.
 * Author : V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 * Version:1.0
 */
@UtilityClass
public class ErrorConstant {

    public static final String MID = "MID";

    public static final String CORRELATION_ID =
            "CORRELATION_ID";

    public static final String REMARK = "REMARK";

    public static final String CREATED_BY =
            "CREATED_BY";

    public static final int QUEUE_SIZE = 10000;

    public static final int BATCH_SIZE = 50;
}

====================================================

4. TABLE COLUMN REMOVE

REMOVE THESE COLUMNS FROM TABLE:

1. ORDER_REF_NUM
2. ATRN_NUM
3. PAY_MODE

====================================================

5. REMOVE FILES

DELETE THESE FILES:

1. ExceptionCleanupScheduler.java
2. ExceptionPartitionCleanupDao.java
3. ExceptionPartitionCleanupService.java
4. ExceptionTrackerQuery.java

====================================================

6. KEEP THESE FILES AS IT IS

NO CHANGE:

1. ExceptionTrackerAspect.java
2. MDCUtil.java
3. ExceptionUtil.java
4. StackTraceUtil.java
5. ExceptionLogMapper.java
6. ExceptionLogRepository.java
7. ExceptionLogDto.java
8. ExceptionLog.java
9. TrackException.java
10. ExceptionTaskExecutorConfig.java


package com.sbi.epay.exceptionTracker.repository;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.entity.ExceptionLog;
import com.sbi.epay.exceptionTracker.mapper.ExceptionLogMapper;
import com.sbi.epay.exceptionTracker.util.ErrorConstant;
import com.sbi.epay.exceptionTracker.util.MDCUtil;
import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;
import jakarta.annotation.PostConstruct;
import org.apache.commons.lang3.ObjectUtils;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.core.task.TaskExecutor;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

@Component
public class ExceptionBufferedRepository {

    private static final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(ExceptionBufferedRepository.class);

    private final ExceptionLogRepository repository;
    private final ExceptionLogMapper mapper;
    private final ObjectMapper objectMapper;
    private final TaskExecutor taskExecutor;

    private final BlockingQueue<ExceptionLog> queue =
            new LinkedBlockingQueue<>(ErrorConstant.QUEUE_SIZE);

    public ExceptionBufferedRepository(
            ExceptionLogRepository repository,
            ExceptionLogMapper mapper,
            ObjectMapper objectMapper,
            @Qualifier("exceptionTaskExecutor") TaskExecutor taskExecutor
    ) {
        this.repository = repository;
        this.mapper = mapper;
        this.objectMapper = objectMapper;
        this.taskExecutor = taskExecutor;
    }

    @PostConstruct
    public void init() {
        taskExecutor.execute(this::saveAndFlush);
    }

    public void buffer(ExceptionLogDto dto) {

        try {

            prepare(dto);

            ExceptionLog entity = mapper.toEntity(dto);

            if (!queue.offer(entity)) {
                logger.error("Error while buffering exception log, queue full");
            }

        } catch (Exception ex) {
            logger.error("Error while preparing exception log", ex);
        }
    }

    private void saveAndFlush() {

        while (true) {

            try {

                List<ExceptionLog> batch = new ArrayList<>();

                ExceptionLog first = queue.poll(1, TimeUnit.SECONDS);

                if (ObjectUtils.isNotEmpty(first)) {

                    batch.add(first);

                    queue.drainTo(batch, ErrorConstant.BATCH_SIZE - 1);

                    repository.saveAll(batch);

                    logger.info(
                            "Exception log batch saved successfully, batch size : {}",
                            batch.size()
                    );
                }

            } catch (Exception ex) {
                logger.error("Error while saving exception log batch", ex);
            }
        }
    }

    private void prepare(ExceptionLogDto dto) throws Exception {

        Map<String, String> mdc = dto.getMdcMap();

        if (mdc == null) {
            mdc = new HashMap<>();
        }

        dto.setMerchantId(MDCUtil.getIgnoreCase(mdc, ErrorConstant.MID));
        dto.setOrderRefNumber(MDCUtil.getIgnoreCase(mdc, ErrorConstant.ORDER_REF));
        dto.setAtrnNum(MDCUtil.getIgnoreCase(mdc, ErrorConstant.ATRN));
        dto.setPayMode(MDCUtil.getIgnoreCase(mdc, ErrorConstant.PAYMODE));
        dto.setCorrelationId(MDCUtil.getIgnoreCase(mdc, ErrorConstant.CORRELATION_ID));
        dto.setRemark(MDCUtil.getIgnoreCase(mdc, ErrorConstant.REMARK));
        dto.setCreatedBy(MDCUtil.getIgnoreCase(mdc, ErrorConstant.CREATED_BY));
        dto.setCreatedDate(System.currentTimeMillis());
        dto.setMdcJson(objectMapper.writeValueAsString(mdc));
    }
}
#####

package com.sbi.epay.exceptionTracker.repository;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.entity.ExceptionLog;
import com.sbi.epay.exceptionTracker.mapper.ExceptionLogMapper;
import com.sbi.epay.exceptionTracker.util.ErrorConstant;
import com.sbi.epay.exceptionTracker.util.MDCUtil;
import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;
import jakarta.annotation.PostConstruct;
import org.apache.commons.lang3.ObjectUtils;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.core.task.TaskExecutor;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

@Component
public class ExceptionBufferedRepository {

    private static final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(ExceptionBufferedRepository.class);

    private final ExceptionLogRepository repository;
    private final ExceptionLogMapper mapper;
    private final ObjectMapper objectMapper;
    private final TaskExecutor taskExecutor;

    private final BlockingQueue<ExceptionLog> queue =
            new LinkedBlockingQueue<>(ErrorConstant.QUEUE_SIZE);

    public ExceptionBufferedRepository(
            ExceptionLogRepository repository,
            ExceptionLogMapper mapper,
            ObjectMapper objectMapper,
            @Qualifier("exceptionTaskExecutor") TaskExecutor taskExecutor
    ) {
        this.repository = repository;
        this.mapper = mapper;
        this.objectMapper = objectMapper;
        this.taskExecutor = taskExecutor;
    }

    @PostConstruct
    public void init() {
        taskExecutor.execute(this::saveAndFlush);
    }

    public void buffer(ExceptionLogDto dto) {

        try {

            prepare(dto);

            ExceptionLog entity = mapper.toEntity(dto);

            if (!queue.offer(entity)) {
                logger.error("Error while buffering exception log, queue full");
            }

        } catch (Exception ex) {
            logger.error("Error while preparing exception log", ex);
        }
    }

    private void saveAndFlush() {

        while (true) {

            try {

                List<ExceptionLog> batch = new ArrayList<>();

                ExceptionLog first = queue.poll(1, TimeUnit.SECONDS);

                if (ObjectUtils.isNotEmpty(first)) {

                    batch.add(first);

                    queue.drainTo(batch, ErrorConstant.BATCH_SIZE - 1);

                    repository.saveAll(batch);

                    logger.info(
                            "Exception log batch saved successfully, batch size : {}",
                            batch.size()
                    );
                }

            } catch (Exception ex) {
                logger.error("Error while saving exception log batch", ex);
            }
        }
    }

    private void prepare(ExceptionLogDto dto) throws Exception {

        Map<String, String> mdc = dto.getMdcMap();

        if (mdc == null) {
            mdc = new HashMap<>();
        }

        dto.setMerchantId(MDCUtil.getIgnoreCase(mdc, ErrorConstant.MID));
        dto.setOrderRefNumber(MDCUtil.getIgnoreCase(mdc, ErrorConstant.ORDER_REF));
        dto.setAtrnNum(MDCUtil.getIgnoreCase(mdc, ErrorConstant.ATRN));
        dto.setPayMode(MDCUtil.getIgnoreCase(mdc, ErrorConstant.PAYMODE));
        dto.setCorrelationId(MDCUtil.getIgnoreCase(mdc, ErrorConstant.CORRELATION_ID));
        dto.setRemark(MDCUtil.getIgnoreCase(mdc, ErrorConstant.REMARK));
        dto.setCreatedBy(MDCUtil.getIgnoreCase(mdc, ErrorConstant.CREATED_BY));
        dto.setCreatedDate(System.currentTimeMillis());
        dto.setMdcJson(objectMapper.writeValueAsString(mdc));
    }
}
#########
package com.sbi.epay.exceptionTracker.repository;

import com.fasterxml.jackson.databind.ObjectMapper;

import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.entity.ExceptionLog;
import com.sbi.epay.exceptionTracker.mapper.ExceptionLogMapper;
import com.sbi.epay.exceptionTracker.util.ErrorConstant;
import com.sbi.epay.exceptionTracker.util.MDCUtil;

import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;

import jakarta.annotation.PostConstruct;

import org.apache.commons.lang3.ObjectUtils;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.core.task.TaskExecutor;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

/**
 * Class Name : ExceptionBufferedRepository
 * Description : Buffered repository used for asynchronous
 * batch processing of exception logs.
 * Author : V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * All rights reserved
 *
 * Version:1.0
 */
@Component
public class ExceptionBufferedRepository {

    private static final LoggerUtility logger = LoggerFactoryUtility.getLogger(ExceptionBufferedRepository.class);
    private final ExceptionLogRepository repository;
    private final ExceptionLogMapper mapper;
    private final ObjectMapper objectMapper;
    private final TaskExecutor taskExecutor;
    private final BlockingQueue<ExceptionLogDto> queue = new LinkedBlockingQueue<>(ErrorConstant.QUEUE_SIZE);

    public ExceptionBufferedRepository(
            ExceptionLogRepository repository,
            ExceptionLogMapper mapper,
            ObjectMapper objectMapper,
            @Qualifier("exceptionTaskExecutor")
            TaskExecutor taskExecutor
    ) {

        this.repository = repository;
        this.mapper = mapper;
        this.objectMapper = objectMapper;
        this.taskExecutor = taskExecutor;
    }

    @PostConstruct
    public void init() {
        taskExecutor.execute(this::saveAndFlush);
    }

    public void buffer(ExceptionLogDto dto) {

        if (!queue.offer(dto)) {
            logger.error("Error while buffering exception log, queue full");
        }
    }
    private void saveAndFlush() {

        while (true) {

            try {

                List<ExceptionLog> batch = new ArrayList<>();
                ExceptionLogDto first = queue.poll(1, TimeUnit.SECONDS);
                if (ObjectUtils.isNotEmpty(first)) {
                    try {
                        prepare(first);
                        batch.add(mapper.toEntity(first));

                    } catch (Exception ex) {
                        logger.error("Error while preparing first exception log", ex);
                    }

                    List<ExceptionLogDto> temp = new ArrayList<>();
                    queue.drainTo(temp, ErrorConstant.BATCH_SIZE);

                    for (ExceptionLogDto dto : temp) {

                        try {
                            prepare(dto);
                            batch.add(mapper.toEntity(dto));

                        } catch (Exception ex) {
                            logger.error("Error while preparing exception log", ex);
                        }
                    }
                    if (!batch.isEmpty()) {
                        repository.saveAll(batch);
                        logger.info("Exception log batch saved successfully, batch size : {}", batch.size()
                        );
                    }
                }
            } catch (Exception ex) {
                logger.error("Error while saving exception log batch", ex);
            }
        }
    }

    private void prepare(ExceptionLogDto dto) throws Exception {

        Map<String, String> mdc = dto.getMdcMap();
        if (mdc == null) {
            mdc = new HashMap<>();
        }

        dto.setMerchantId(MDCUtil.getIgnoreCase(mdc, ErrorConstant.MID));
        dto.setOrderRefNumber(MDCUtil.getIgnoreCase(mdc, ErrorConstant.ORDER_REF));
        dto.setAtrnNum(MDCUtil.getIgnoreCase(mdc, ErrorConstant.ATRN));
        dto.setPayMode(MDCUtil.getIgnoreCase(mdc, ErrorConstant.PAYMODE));
        dto.setCorrelationId(MDCUtil.getIgnoreCase(mdc, ErrorConstant.CORRELATION_ID));
        dto.setRemark(MDCUtil.getIgnoreCase(mdc, ErrorConstant.REMARK));
        dto.setCreatedBy(MDCUtil.getIgnoreCase(mdc, ErrorConstant.CREATED_BY));
        dto.setCreatedDate(System.currentTimeMillis());
        dto.setMdcJson(objectMapper.writeValueAsString(mdc));
    }
}

###############################################
========================================
FINAL ExceptionTaskExecutorConfig.java

package com.sbi.epay.exceptionTracker.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

/**

* Class Name : ExceptionTaskExecutorConfig

* Description : Configuration class used

* to create async task executor bean.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
  @Configuration
  public class ExceptionTaskExecutorConfig {
  
  /**
  
  * Creates task executor bean used for
  
  * asynchronous queue processing.
    */
    @Bean(name = "exceptionTaskExecutor")
    public ThreadPoolTaskExecutor exceptionTaskExecutor() {
    
    ThreadPoolTaskExecutor executor =
    new ThreadPoolTaskExecutor();
    
    /**
    
    * Minimum active threads.
      */
      executor.setCorePoolSize(2);
    
    /**
    
    * Maximum threads allowed.
      */
      executor.setMaxPoolSize(5);
    
    /**
    
    * Queue capacity.
      */
      executor.setQueueCapacity(1000);
    
    /**
    
    * Thread naming pattern.
      */
      executor.setThreadNamePrefix(
      "exception-tracker-"
      );
    
    /**
    
    * Initializes executor.
      */
      executor.initialize();
    
    return executor;
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
  
  * Queue used for storing entities.
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
    
    taskExecutor.execute(
    this::saveAndFlush
    );
    }
  
  /**
  
  * Adds entity into queue.
    */
    public void buffer(
    ExceptionLog entity
    ) {
    
    if (!queue.offer(entity)) {
    
     logger.error(
         "Error while buffering exception log, queue full"
 );
    
    }
    }
  
  /**
  
  * Consumes queue data and performs
  
  * direct database bulk save.
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
          * Direct DB bulk save.
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

========================================
FINAL ExceptionLogRepository.java

package com.sbi.epay.exceptionTracker.repository;

import com.sbi.epay.exceptionTracker.entity.ExceptionLog;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

/**

* Class Name : ExceptionLogRepository
* Description : Repository interface used
* for ExceptionLog database operations.
* 
* Author : V1024113(Rohit Gardi)
* Copyright (c) 2025 [State Bank of India]
* All rights reserved
* 
* Version:1.0
  */
  @Repository
  public interface ExceptionLogRepository
  extends JpaRepository<ExceptionLog, Long> {

}

========================================
FINAL ExceptionLogDto.java

package com.sbi.epay.exceptionTracker.dto;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import java.util.Map;

/**

* Class Name : ExceptionLogDto

* Description : DTO class used to transfer

* exception log details.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
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
  
  private String correlationId;
  
  private String remark;
  
  private String mdcJson;
  
  private String createdBy;
  
  private Long createdDate;
  
  private Map<String, String> mdcMap;
  }

========================================
FINAL ExceptionLogMapper.java

package com.sbi.epay.exceptionTracker.mapper;

import com.sbi.epay.exceptionTracker.dto.ExceptionLogDto;
import com.sbi.epay.exceptionTracker.entity.ExceptionLog;

import org.mapstruct.Mapper;

/**

* Class Name : ExceptionLogMapper

* Description : Mapper interface used for

* DTO and Entity conversion.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
  @Mapper(componentModel = "spring")
  public interface ExceptionLogMapper {
  
  /**
  
  * Converts DTO into Entity.
    */
    ExceptionLog toEntity(
    ExceptionLogDto dto
    );
    }

========================================
FINAL TrackException.java

package com.sbi.epay.exceptionTracker.annotation;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.annotation.ElementType;
import java.lang.annotation.Documented;

/**

* Class Name : TrackException

* Description : Annotation used to track

* exceptions for methods.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  public @interface TrackException {
  
  /**
  
  * Stores stack trace if enabled.
    */
    boolean storeStackTrace() default true;
  
  /**
  
  * Rethrows exception if enabled.
    */
    boolean rethrow() default true;
    }

========================================
FINAL ExceptionTaskExecutorConfig.java

package com.sbi.epay.exceptionTracker.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;

/**

* Class Name : ExceptionTaskExecutorConfig

* Description : Configuration class used

* to create async task executor.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
  @Configuration
  public class ExceptionTaskExecutorConfig {
  
  /**
  
  * Task executor bean for async processing.
    */
    @Bean(name = "exceptionTaskExecutor")
    public Executor exceptionTaskExecutor() {
    
    ThreadPoolTaskExecutor executor =
    new ThreadPoolTaskExecutor();
    
    executor.setCorePoolSize(2);
    executor.setMaxPoolSize(5);
    executor.setQueueCapacity(1000);
    executor.setThreadNamePrefix(
    "exception-tracker-"
    );
    
    executor.initialize();
    
    return executor;
    }
    }


    ========================================
FINAL 101_create_table_exception_log.sql

-- LIQUIBASE FORMATTED SQL

-- CHANGESSET RohitG:1

CREATE TABLE EXCEPTION_LOG (

ID NUMBER GENERATED BY DEFAULT AS IDENTITY,

SERVICE_NAME VARCHAR2(50),

CLASS_NAME VARCHAR2(100),

METHOD_NAME VARCHAR2(100),

EXCEPTION_TYPE VARCHAR2(100),

EXCEPTION_MESSAGE CLOB,

STACK_TRACE CLOB,

MERCHANT_ID VARCHAR2(50),

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
    +
    NUMTODSINTERVAL(
            CREATED_DATE / 1000,
            'SECOND'
    )
) VIRTUAL

)

PCTFREE 30
INITRANS 100
MAXTRANS 255

PARTITION BY RANGE (PARTITION_DATE)

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

========================================
REMOVED FROM TABLE

1. ORDER_REF_NUMBER

2. ATRN_NUM

3. PAY_MODE

========================================
FINAL ExceptionLogDao.java

package com.sbi.epay.exceptionTracker.dao;

import com.sbi.epay.exceptionTracker.entity.ExceptionLog;
import com.sbi.epay.exceptionTracker.repository.ExceptionLogRepository;

import lombok.RequiredArgsConstructor;

import org.springframework.stereotype.Component;

import java.util.List;

/**

* Class Name : ExceptionLogDao

* Description : DAO class used for

* exception log database operations.

* 

* Author : V1024113(Rohit Gardi)

* Copyright (c) 2025 [State Bank of India]

* All rights reserved

* 

* Version:1.0
  */
  @Component
  @RequiredArgsConstructor
  public class ExceptionLogDao {
  
  private final ExceptionLogRepository repository;
  
  /**
  
  * Saves exception log batch.
    */
    public void saveAll(
    List<ExceptionLog> logs
    ) {
    
    repository.saveAll(logs);
    }
    }

========================================
FINAL build.gradle

dependencies {

implementation 'org.springframework.boot:spring-boot-starter-aop'

implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

implementation 'org.springframework.boot:spring-boot-starter-web'

implementation 'org.mapstruct:mapstruct:1.5.5.Final'

annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.5.Final'

implementation 'org.projectlombok:lombok'

annotationProcessor 'org.projectlombok:lombok'

implementation 'com.fasterxml.jackson.core:jackson-databind'

}

========================================
REMOVE COMPLETE

REMOVE THESE FROM ENTIRE PROJECT:

1. ORDER_REF

2. ATRN

3. PAYMODE

4. ORDER_REF_NUMBER

5. ATRN_NUM

6. PAY_MODE

7. prepare()

8. prepare(dto)

9. mapper.toEntity()

10. dto.setOrderRefNumber()

11. dto.setAtrnNum()

12. dto.setPayMode()

13. ExceptionPartitionCleanupDao

14. ExceptionPartitionCleanupService

15. ExceptionCleanupScheduler

16. ExceptionTrackerQuery

    
  


    
