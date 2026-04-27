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
// STEP 1 — DB TABLE (RUN IN DATABASE)
// ============================================================

/*
CREATE TABLE exception_log (
    type        VARCHAR(255),
    stacktrace  VARCHAR(5000),
    path        VARCHAR(500),
    merchant_id VARCHAR(100),
    remark      VARCHAR(1000),
    created_at  BIGINT,
    created_by  VARCHAR(100)
);
*/


// ============================================================
// STEP 2 — ENTITY CLASS
// FILE: com.epay.merchant.entity.ExceptionLog.java
// ============================================================

package com.epay.merchant.entity;

import jakarta.persistence.*;

@Entity
@Table(name = "exception_log")
public class ExceptionLog {

    @Id
    private Long createdAt; // JPA la ID pahije mhanun use karto

    private String type;

    @Column(length = 5000)
    private String stacktrace;

    private String path;

    private String merchantId;

    private String remark;

    private String createdBy;

    // getters setters

    public Long getCreatedAt() { return createdAt; }
    public void setCreatedAt(Long createdAt) { this.createdAt = createdAt; }

    public String getType() { return type; }
    public void setType(String type) { this.type = type; }

    public String getStacktrace() { return stacktrace; }
    public void setStacktrace(String stacktrace) { this.stacktrace = stacktrace; }

    public String getPath() { return path; }
    public void setPath(String path) { this.path = path; }

    public String getMerchantId() { return merchantId; }
    public void setMerchantId(String merchantId) { this.merchantId = merchantId; }

    public String getRemark() { return remark; }
    public void setRemark(String remark) { this.remark = remark; }

    public String getCreatedBy() { return createdBy; }
    public void setCreatedBy(String createdBy) { this.createdBy = createdBy; }
}


// ============================================================
// STEP 3 — REPOSITORY
// FILE: com.epay.merchant.repository.ExceptionLogRepository.java
// ============================================================

package com.epay.merchant.repository;

import com.epay.merchant.entity.ExceptionLog;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ExceptionLogRepository extends JpaRepository<ExceptionLog, Long> {
}


// ============================================================
// STEP 4 — SERVICE
// FILE: com.epay.merchant.service.ExceptionLogService.java
// ============================================================

package com.epay.merchant.service;

import com.epay.merchant.entity.ExceptionLog;
import com.epay.merchant.repository.ExceptionLogRepository;
import org.springframework.stereotype.Service;

import java.io.PrintWriter;
import java.io.StringWriter;

@Service
public class ExceptionLogService {

    private final ExceptionLogRepository repository;

    public ExceptionLogService(ExceptionLogRepository repository) {
        this.repository = repository;
    }

    public void logException(Exception e, String merchantId, String remark) {

        try {
            ExceptionLog log = new ExceptionLog();

            log.setCreatedAt(System.currentTimeMillis()); // unique id
            log.setType(e.getClass().getName());
            log.setStacktrace(getStackTrace(e));
            log.setPath(getPath(e));
            log.setMerchantId(merchantId != null ? merchantId : "SYSTEM");
            log.setRemark(remark);
            log.setCreatedBy("SYSTEM");

            repository.save(log);

        } catch (Exception ex) {
            ex.printStackTrace(); // fallback
        }
    }

    private String getStackTrace(Exception e) {
        StringWriter sw = new StringWriter();
        e.printStackTrace(new PrintWriter(sw));
        return sw.toString();
    }

    private String getPath(Exception e) {
        StackTraceElement element = e.getStackTrace()[0];
        return element.getClassName() + "." + element.getMethodName()
                + "(" + element.getFileName() + ":" + element.getLineNumber() + ")";
    }
}


// ============================================================
// STEP 5 — HOW TO USE (ANY EXISTING CLASS)
// ============================================================

/*

// inject service
private final ExceptionLogService exceptionLogService;

public YourClass(ExceptionLogService exceptionLogService) {
    this.exceptionLogService = exceptionLogService;
}


// use in catch block
try {
    // your logic
} catch (Exception e) {

    exceptionLogService.logException(e, merchantId, "your method failed");

    throw e;
}

*/


// ============================================================
// 🎯 FINAL RESULT
// ============================================================

/*
✔ Exception DB madhe store honar
✔ Stacktrace full milnar
✔ Exact line (path) milnar
✔ Debugging easy honar
✔ Global handler nahi use kela (as per requirement)

*/




package com.epay.merchant.service;

import com.epay.merchant.dao.ExceptionLogDao;
import com.epay.merchant.dto.ExceptionLogDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.io.PrintWriter;
import java.io.StringWriter;

@Service
@RequiredArgsConstructor
public class ExceptionLogService {

    private final ExceptionLogDao exceptionLogDao;

    public void logException(Exception e, String merchantId, String remark) {

        try {
            ExceptionLogDto dto = ExceptionLogDto.builder()
                    .type(e.getClass().getName())
                    .stacktrace(getStackTrace(e))
                    .path(getRootPath(e))
                    .merchantId(merchantId != null ? merchantId : "SYSTEM")
                    .remark(remark != null ? remark : e.getMessage())
                    .createdAt(System.currentTimeMillis())
                    .createdBy(merchantId != null ? merchantId : "SYSTEM")
                    .build();

            exceptionLogDao.saveExceptionLog(dto);

        } catch (Exception ex) {
            System.err.println("ExceptionLogService failed: " + ex.getMessage());
        }
    }

    private String getStackTrace(Exception e) {
        StringWriter sw = new StringWriter();
        e.printStackTrace(new PrintWriter(sw));
        String trace = sw.toString();

        return trace.length() > 5000 ? trace.substring(0, 5000) : trace;
    }

    private String getRootPath(Exception e) {

        Throwable root = e;

        while (root.getCause() != null) {
            root = root.getCause();
        }

        StackTraceElement[] stackTrace = root.getStackTrace();

        if (stackTrace == null || stackTrace.length == 0) {
            return "unknown";
        }

        StackTraceElement element = stackTrace[0];

        return element.getClassName() + "."
                + element.getMethodName()
                + "(" + element.getFileName()
                + ":" + element.getLineNumber() + ")";
    }
}
