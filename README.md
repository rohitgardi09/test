package com.epay.merchant.service;

import com.epay.merchant.dao.ExceptionLogDao;
import com.epay.merchant.dto.ExceptionLogDto;
import lombok.RequiredArgsConstructor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;
import org.springframework.stereotype.Service;
import org.springframework.util.ObjectUtils;
import org.springframework.util.StringUtils;

import java.io.PrintWriter;
import java.io.StringWriter;

@Service
@RequiredArgsConstructor
public class ExceptionLogService {

    private static final Logger log = LoggerFactory.getLogger(ExceptionLogService.class);

    private final ExceptionLogDao exceptionLogDao;

    public void logException(Exception e, String merchantId, String remark) {
        try {
            ExceptionLogDto dto = buildExceptionLogDto(e, merchantId, remark);
            exceptionLogDao.saveExceptionLog(dto);
        } catch (Exception ex) {
            log.error("Failed to persist exception log", ex);
        }
    }

    private ExceptionLogDto buildExceptionLogDto(Exception e, String merchantId, String remark) {
        return ExceptionLogDto.builder()
                .type(e.getClass().getName())
                .stacktrace(getStackTrace(e))
                .path(getRootPath(e))
                .merchantId(StringUtils.hasText(merchantId) ? merchantId : "NA")
                .remark(StringUtils.hasText(remark) ? remark : e.getMessage())
                .correlationId(getCorrelationId())
                .createdAt(System.currentTimeMillis())
                .createdBy(StringUtils.hasText(merchantId) ? merchantId : "SYSTEM")
                .build();
    }

    private String getCorrelationId() {
        String correlationId = MDC.get("correlationId");
        return StringUtils.hasText(correlationId) ? correlationId : "NA";
    }

    private String getStackTrace(Exception e) {
        StringWriter stringWriter = new StringWriter();
        PrintWriter printWriter = new PrintWriter(stringWriter);
        e.printStackTrace(printWriter);
        return stringWriter.toString();
    }

    private String getRootPath(Exception e) {
        Throwable root = e;
        while (root.getCause() != null) {
            root = root.getCause();
        }

        StackTraceElement[] stackTrace = root.getStackTrace();

        if (ObjectUtils.isEmpty(stackTrace)) {
            return "unknown";
        }

        StackTraceElement element = stackTrace[0];

        return element.getClassName() + "."
                + element.getMethodName()
                + "(line:" + element.getLineNumber() + ")";
    }
}
