// ════════════════════════════════════════════════════════════════════════════
// FILE 1: src/main/java/com/epay/merchant/annotation/FutureDeployment.java
// ════════════════════════════════════════════════════════════════════════════

package com.epay.merchant.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Controls whether a future-deployment API is available for execution.
 *
 * Usage:
 *   @FutureDeployment(enabled = true)  → API is accessible
 *   @FutureDeployment(enabled = false) → API returns 404
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface FutureDeployment {
    boolean enabled() default false; // safe by default — disabled jara specify nahi kela tar
}


// ════════════════════════════════════════════════════════════════════════════
// FILE 2: src/main/java/com/epay/merchant/util/ErrorConstants.java
// ════════════════════════════════════════════════════════════════════════════

package com.epay.merchant.util;

/**
 * Centralized error constants for FutureDeployment API responses.
 * Sagle hardcoded strings eka jaghi — easy to maintain & update.
 */
public final class ErrorConstants {

    private ErrorConstants() {} // utility class — instantiate nahi karnar

    // ─── HTTP Status ─────────────────────────────────────────────────────
    public static final int    STATUS_NOT_FOUND   = 404;

    // ─── Response Field Keys ─────────────────────────────────────────────
    public static final String KEY_STATUS         = "status";
    public static final String KEY_ERROR          = "error";
    public static final String KEY_MESSAGE        = "message";
    public static final String KEY_PATH           = "path";
    public static final String KEY_TIMESTAMP      = "timestamp";

    // ─── Response Field Values ────────────────────────────────────────────
    public static final String ERROR_NOT_FOUND    = "Not Found";
    public static final String MSG_API_DISABLED   =
            "This API is not available in the current deployment.";
}


// ════════════════════════════════════════════════════════════════════════════
// FILE 3: src/main/java/com/epay/merchant/exception/ApiDisabledException.java
// ════════════════════════════════════════════════════════════════════════════

package com.epay.merchant.exception;

public class ApiDisabledException extends RuntimeException {

    private final String apiPath;

    public ApiDisabledException(String apiPath) {
        super("API is currently disabled for this deployment: " + apiPath);
        this.apiPath = apiPath;
    }

    public String getApiPath() {
        return apiPath;
    }
}


// ════════════════════════════════════════════════════════════════════════════
// FILE 4: src/main/java/com/epay/merchant/config/FutureDeploymentAspect.java
// ════════════════════════════════════════════════════════════════════════════

package com.epay.merchant.config;

import com.epay.merchant.annotation.FutureDeployment;
import com.epay.merchant.exception.ApiDisabledException;
import jakarta.servlet.http.HttpServletRequest;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import java.lang.reflect.Method;

@Aspect
@Component
public class FutureDeploymentAspect {

    /**
     * Intercepts ALL methods/classes annotated with @FutureDeployment
     */
    @Around("@annotation(com.epay.merchant.annotation.FutureDeployment) || " +
            "@within(com.epay.merchant.annotation.FutureDeployment)")
    public Object checkFutureDeployment(ProceedingJoinPoint joinPoint) throws Throwable {

        FutureDeployment annotation = resolveAnnotation(joinPoint);

        if (annotation != null && !annotation.enabled()) {
            String path = getCurrentRequestPath();
            throw new ApiDisabledException(path);
        }

        return joinPoint.proceed(); // ✅ enabled ahe — controller execute hoel
    }

    /**
     * Method-level annotation > Class-level annotation (priority)
     */
    private FutureDeployment resolveAnnotation(ProceedingJoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();

        // 1. Method level first (highest priority)
        FutureDeployment methodAnnotation = method.getAnnotation(FutureDeployment.class);
        if (methodAnnotation != null) return methodAnnotation;

        // 2. Class level fallback
        return joinPoint.getTarget()
                        .getClass()
                        .getAnnotation(FutureDeployment.class);
    }

    private String getCurrentRequestPath() {
        try {
            ServletRequestAttributes attrs =
                (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            if (attrs != null) {
                return attrs.getRequest().getRequestURI();
            }
        } catch (Exception ignored) {}
        return "unknown";
    }
}


// ════════════════════════════════════════════════════════════════════════════
// FILE 5: src/main/java/com/epay/merchant/exceptionhandlers/GlobalExceptionHandler.java
// ════════════════════════════════════════════════════════════════════════════

package com.epay.merchant.exceptionhandlers;

import com.epay.merchant.exception.ApiDisabledException;
import com.epay.merchant.util.ErrorConstants;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.Instant;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ApiDisabledException.class)
    public ResponseEntity<Map<String, Object>> handleApiDisabled(ApiDisabledException ex) {
        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(Map.of(
                ErrorConstants.KEY_STATUS,    ErrorConstants.STATUS_NOT_FOUND,
                ErrorConstants.KEY_ERROR,     ErrorConstants.ERROR_NOT_FOUND,
                ErrorConstants.KEY_MESSAGE,   ErrorConstants.MSG_API_DISABLED,
                ErrorConstants.KEY_PATH,      ex.getApiPath(),
                ErrorConstants.KEY_TIMESTAMP, Instant.now().toString()
            ));
    }
}


// ════════════════════════════════════════════════════════════════════════════
// FILE 6: USAGE EXAMPLE — src/main/java/com/epay/merchant/controller/
// ════════════════════════════════════════════════════════════════════════════

package com.epay.merchant.controller;

import com.epay.merchant.annotation.FutureDeployment;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/merchant")
public class MerchantController {

    // ✅ Enabled — QA test karu shaktat
    @GetMapping("/feature-new")
    @FutureDeployment(enabled = true)
    public ResponseEntity<String> newFeature() {
        return ResponseEntity.ok("New feature response");
    }

    // 🔒 Disabled — 404 return hoel
    @GetMapping("/feature-wip")
    @FutureDeployment(enabled = false)
    public ResponseEntity<String> wipFeature() {
        return ResponseEntity.ok("Never reached");
    }
}


// ════════════════════════════════════════════════════════════════════════════
// build.gradle — Required Dependency (already nahi asel tar add kara)
// ════════════════════════════════════════════════════════════════════════════

// implementation 'org.springframework.boot:spring-boot-starter-aop'
