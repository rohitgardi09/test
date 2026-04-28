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
                    .merchantId(merchantId != null ? merchantId : "NA")
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


1.capture and store correlation id also
2.extract this lines and create private method , buildExceptionLogDto
  try {
            ExceptionLogDto dto = ExceptionLogDto.builder()
                    .type(e.getClass().getName())
                    .stacktrace(getStackTrace(e))
                    .path(getRootPath(e))
                    .merchantId(merchantId != null ? merchantId : "NA")
                    .remark(remark != null ? remark : e.getMessage())
                    .createdAt(System.currentTimeMillis())
                    .createdBy(merchantId != null ? merchantId : "SYSTEM")
                    .build();

            exceptionLogDao.saveExceptionLog(dto);

        } catch (Exception ex) {
            System.err.println("ExceptionLogService failed: " + ex.getMessage());
        }
3.use NA if not present
  .merchantId(merchantId != null ? merchantId : "SYSTEM")

4.use StringUtils to check null or empty
    .remark(remark != null ? remark : e.getMessage())

5.optimize this, remove new keyword and length code

 private String getStackTrace(Exception e) {
        StringWriter sw = new StringWriter();
        e.printStackTrace(new PrintWriter(sw));
        String trace = sw.toString();
        return trace.length() > 5000 ? trace.substring(0, 5000) : trace;
    }

6.use objecutils to check empty or null and remove file name
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
