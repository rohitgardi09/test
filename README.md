
api.logging.supported-content-types:application/json,text/plain,application/xml,text/xml





package com.epay.cs.filter;

import com.epay.cs.constant.ApiLogConstants;
import com.epay.cs.service.ApiRequestResponseLogService;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;
import org.springframework.web.util.ContentCachingRequestWrapper;
import org.springframework.web.util.ContentCachingResponseWrapper;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Arrays;
import java.util.UUID;

@Component
@RequiredArgsConstructor
@Slf4j
public class ApiRequestResponseLogFilter
        extends OncePerRequestFilter {

    private final ApiRequestResponseLogService logService;

    @Value("${api.logging.supported-content-types}")
    private String supportedContentTypes;

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain)
            throws ServletException, IOException {

        ContentCachingRequestWrapper requestWrapper =
                new ContentCachingRequestWrapper(request);

        ContentCachingResponseWrapper responseWrapper =
                new ContentCachingResponseWrapper(response);

        String correlationId =
                request.getHeader(
                        ApiLogConstants.CORRELATION_ID_HEADER
                );

        if (correlationId == null ||
                correlationId.isBlank()) {

            correlationId =
                    UUID.randomUUID().toString();
        }

        String url =
                request.getRequestURI();

        try {

            filterChain.doFilter(
                    requestWrapper,
                    responseWrapper
            );

        } finally {

            /*
             * Request
             */
            String requestContentType =
                    requestWrapper.getContentType();

            if (isSupportedContentType(
                    requestContentType)) {

                String requestBody =
                        getRequest(requestWrapper);

                logService.buffer(
                        correlationId,
                        ApiLogConstants.REQUEST,
                        url,
                        requestBody
                );

            } else {

                log.info(
                        "API request log not saved. " +
                        "Unsupported Content-Type: {}, URL: {}, CorrelationId: {}",
                        requestContentType,
                        url,
                        correlationId
                );
            }

            /*
             * Response
             */
            String responseContentType =
                    responseWrapper.getContentType();

            if (isSupportedContentType(
                    responseContentType)) {

                String responseBody =
                        getResponse(responseWrapper);

                logService.buffer(
                        correlationId,
                        ApiLogConstants.RESPONSE,
                        url,
                        responseBody
                );

            } else {

                log.info(
                        "API response log not saved. " +
                        "Unsupported Content-Type: {}, URL: {}, CorrelationId: {}",
                        responseContentType,
                        url,
                        correlationId
                );
            }

            responseWrapper.copyBodyToResponse();
        }
    }

    private String getRequest(
            ContentCachingRequestWrapper request) {

        byte[] content =
                request.getContentAsByteArray();

        return new String(
                content,
                StandardCharsets.UTF_8
        );
    }

    private String getResponse(
            ContentCachingResponseWrapper response) {

        byte[] content =
                response.getContentAsByteArray();

        return new String(
                content,
                StandardCharsets.UTF_8
        );
    }

    private boolean isSupportedContentType(
            String contentType) {

        if (contentType == null ||
                contentType.isBlank()) {

            return false;
        }

        return Arrays.stream(
                        supportedContentTypes.split(",")
                )
                .map(String::trim)
                .anyMatch(contentType::startsWith);
    }
}
