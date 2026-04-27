Topic - Capture Exception and Store in DB

# Description

We need to create a exception log mechanism which can catch all type of exceptions in the system and log into the db with detailed stacktrace where we can analysis which class method, code line has thrown the exception.

# Data Flow

1. User / Scheduler / kafka will triger java methods
2. Java method will execute the code flow
3. Code flow may throw exceptions in between
4. We need to catch and log that exception 

# Testing Check List

* Testing which covers all main functionalities

# Developer Checklist

* Class Flow

![ClassFlow.png](/-/project/22/uploads/fb6a534f1bf8fe74d293c775fc49de14/ClassFlow.png)

* 90% Unit test case coverage

# Development Steps

1. Create ExeptionLog Service
2. Create log exception method which send Exception object, Mid, String remark(optional)
3. Inject this class in calling classes
4. Call log exception method in every catch block

# Database Tables

* Exception_Log (Append Only table)

  | Column | Type |
  |--------|------|
  | Type | varchar |
  | Stacktrace | varchar |
  | Path | varchar |
  | Merchant_Id | varchar |
  | Remark | varchar |
  | Created_At | number |
  | Created_By | varchar |


# Acceptance Criteria

* Test cases should have 90% coverage




// ============================================================
// 1. ExceptionLog.java (Entity)
// ============================================================
package com.example.exceptionlog.entity;

import jakarta.persistence.*;
import lombok.*;
import java.io.Serializable;
import java.util.Objects;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "exception_log")
@IdClass(ExceptionLogId.class)
public class ExceptionLog {

    @Id
    @Column(name = "merchant_id")
    private String merchantId;

    @Id
    @Column(name = "created_at")
    private Long createdAt;

    @Column(name = "type")
    private String type;

    @Column(name = "stacktrace", columnDefinition = "TEXT")
    private String stacktrace;

    @Column(name = "path")
    private String path;

    @Column(name = "remark")
    private String remark;

    @Column(name = "created_by")
    private String createdBy;
}


// ============================================================
// 2. ExceptionLogId.java (Composite Key)
// ============================================================
package com.example.exceptionlog.entity;

import java.io.Serializable;
import java.util.Objects;

public class ExceptionLogId implements Serializable {

    private String merchantId;
    private Long createdAt;

    public ExceptionLogId() {}

    public ExceptionLogId(String merchantId, Long createdAt) {
        this.merchantId = merchantId;
        this.createdAt  = createdAt;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof ExceptionLogId)) return false;
        ExceptionLogId that = (ExceptionLogId) o;
        return Objects.equals(merchantId, that.merchantId)
                && Objects.equals(createdAt, that.createdAt);
    }

    @Override
    public int hashCode() {
        return Objects.hash(merchantId, createdAt);
    }
}


// ============================================================
// 3. ExceptionLogRepository.java
// ============================================================
package com.example.exceptionlog.repository;

import com.example.exceptionlog.entity.ExceptionLog;
import com.example.exceptionlog.entity.ExceptionLogId;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ExceptionLogRepository
        extends JpaRepository<ExceptionLog, ExceptionLogId> {
}


// ============================================================
// 4. ExceptionLogService.java (Interface)
// ============================================================
package com.example.exceptionlog.service;

public interface ExceptionLogService {
    void logException(Exception exception, String merchantId, String remark);
    void logException(Exception exception, String merchantId);
    void logException(Exception exception);
}


// ============================================================
// 5. ExceptionLogServiceImpl.java
// ============================================================
package com.example.exceptionlog.service.impl;

import com.example.exceptionlog.entity.ExceptionLog;
import com.example.exceptionlog.repository.ExceptionLogRepository;
import com.example.exceptionlog.service.ExceptionLogService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

import java.io.PrintWriter;
import java.io.StringWriter;

@Service
public class ExceptionLogServiceImpl implements ExceptionLogService {

    private static final Logger log = LoggerFactory.getLogger(ExceptionLogServiceImpl.class);
    public static final String SYSTEM = "SYSTEM";

    private final ExceptionLogRepository repository;

    public ExceptionLogServiceImpl(ExceptionLogRepository repository) {
        this.repository = repository;
    }

    @Override
    public void logException(Exception exception, String merchantId, String remark) {
        if (exception == null) {
            log.warn("logException called with null exception – skipping.");
            return;
        }
        try {
            ExceptionLog entry = buildEntry(exception, merchantId, remark);
            repository.save(entry);
        } catch (Exception e) {
            log.error("Failed to persist exception log", e);
        }
    }

    @Override
    public void logException(Exception exception, String merchantId) {
        logException(exception, merchantId, null);
    }

    @Override
    public void logException(Exception exception) {
        logException(exception, SYSTEM, null);
    }

    private ExceptionLog buildEntry(Exception exception, String merchantId, String remark) {
        ExceptionLog entry = new ExceptionLog();
        entry.setType(exception.getClass().getName());
        entry.setStacktrace(extractStackTrace(exception));
        entry.setPath(extractPath(exception));
        entry.setMerchantId(merchantId != null && !merchantId.isBlank() ? merchantId : SYSTEM);
        entry.setRemark(remark);
        entry.setCreatedAt(System.currentTimeMillis());
        entry.setCreatedBy(SYSTEM);
        return entry;
    }

    private String extractStackTrace(Throwable throwable) {
        StringWriter sw = new StringWriter();
        throwable.printStackTrace(new PrintWriter(sw));
        return sw.toString();
    }

    private String extractPath(Throwable throwable) {
        StackTraceElement[] frames = throwable.getStackTrace();
        if (frames == null || frames.length == 0) return "unknown";

        for (StackTraceElement frame : frames) {
            String cls = frame.getClassName();
            if (!cls.startsWith("java.")
                    && !cls.startsWith("javax.")
                    && !cls.startsWith("sun.")
                    && !cls.startsWith("org.springframework.")
                    && !cls.startsWith("org.hibernate.")) {
                return cls + "." + frame.getMethodName()
                        + "(" + frame.getFileName() + ":" + frame.getLineNumber() + ")";
            }
        }

        StackTraceElement first = frames[0];
        return first.getClassName() + "." + first.getMethodName()
                + "(" + first.getFileName() + ":" + first.getLineNumber() + ")";
    }
}


// ============================================================
// 6. DB Table — SQL
// ============================================================

// CREATE TABLE exception_log (
//     type        VARCHAR,
//     stacktrace  TEXT,
//     path        VARCHAR,
//     merchant_id VARCHAR,
//     remark      VARCHAR,
//     created_at  BIGINT,
//     created_by  VARCHAR
// );


// ============================================================
// 7. Tumchya Existing Service madhe Use kara
// ============================================================

// private final ExceptionLogService exceptionLogService;
//
// public YourService(ExceptionLogService exceptionLogService) {
//     this.exceptionLogService = exceptionLogService;
// }
//
// } catch (Exception e) {
//     exceptionLogService.logException(e, merchantId, "method name failed");
// }
