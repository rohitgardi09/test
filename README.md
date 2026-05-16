package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

/**
 * Class Name: ErrorConstant
 * Description: Utility class used to store application constants.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * ALL rights reserved
 *
 * <p>
 * Version:1.0
 **/
@UtilityClass
public class ErrorConstant {

    public static final String MID = "MID";
    public static final String ORDER_REF = "ORDER_REF";
    public static final String ATRN = "ATRN";
    public static final String PAYMODE = "PAYMODE";
    public static final String CORRELATION_ID = "CORRELATION_ID";
    public static final String REMARK = "REMARK";
    public static final String CREATED_BY = "CREATED_BY";

    public static final String DEFAULT_EXCEPTION_MESSAGE =
            "Exception message is not available";

    public static final String DEFAULT_MDC_VALUE =
            "MDC value is not available";

    public static final int STACK_TRACE_LIMIT = 10;
    public static final int QUEUE_SIZE = 10000;
    public static final int BATCH_SIZE = 50;
    public static final int EXCEPTION_LOG_RETENTION_DAYS = 7;
}

package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

/**
 * Class Name: ExceptionUtil
 * Description: Utility class used to extract exception message safely.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * ALL rights reserved
 *
 * <p>
 * Version:1.0
 **/
@UtilityClass
public class ExceptionUtil {

    /**
     * Safely fetches exception message.
     *
     * @param ex exception object
     * @return exception message
     */
    public static String getMessage(Throwable ex) {

        if (ex == null) {
            return ErrorConstant.DEFAULT_EXCEPTION_MESSAGE;
        }

        return ex.getMessage() != null
                ? ex.getMessage()
                : ErrorConstant.DEFAULT_EXCEPTION_MESSAGE;
    }
}

package com.sbi.epay.exceptionTracker.util;

import lombok.experimental.UtilityClass;

import java.util.Map;

/**
 * Class Name: MDCUtil
 * Description: Utility class used to fetch MDC values safely.
 * Author: V1024113(Rohit Gardi)
 * Copyright (c) 2025 [State Bank of India]
 * ALL rights reserved
 *
 * <p>
 * Version:1.0
 **/
@UtilityClass
public class MDCUtil {

    /**
     * Fetches MDC value ignoring key case.
     *
     * @param map  MDC map
     * @param keys possible keys
     * @return MDC value
     */
    public static String getIgnoreCase(
            Map<String, String> map,
            String... keys) {

        if (map == null) {
            return ErrorConstant.DEFAULT_MDC_VALUE;
        }

        for (String key : keys) {

            for (Map.Entry<String, String> entry : map.entrySet()) {

                if (entry.getKey().equalsIgnoreCase(key)) {

                    return entry.getValue() != null
                            ? entry.getValue()
                            : ErrorConstant.DEFAULT_MDC_VALUE;
                }
            }
        }

        return ErrorConstant.DEFAULT_MDC_VALUE;
    }
}
