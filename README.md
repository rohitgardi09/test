// ============================================================
// FILE 1: dto/admin/UserSearchDto.java
// ============================================================

package com.epay.admin.portal.dto.admin;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserSearchDto {
    private String adId;
    private String name;
    private String emailId;
    private String phoneNumber;
}


// ============================================================
// FILE 2: service/admin/LdapUserSearchService.java
// ============================================================

package com.epay.admin.portal.service.admin;

import com.epay.admin.portal.dto.admin.UserSearchDto;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.ldap.core.AttributesMapper;
import org.springframework.ldap.core.LdapTemplate;
import org.springframework.ldap.filter.AndFilter;
import org.springframework.ldap.filter.EqualsFilter;
import org.springframework.ldap.filter.OrFilter;
import org.springframework.ldap.filter.WhitespaceWildcardsFilter;
import org.springframework.stereotype.Service;

import javax.naming.NamingException;
import javax.naming.directory.Attributes;
import javax.naming.directory.SearchControls;
import java.util.List;

@Service
public class LdapUserSearchService {

    private final LdapTemplate ldapTemplate;

    @Value("${auth.ldap.user-search-base}")
    private String userSearchBase;

    public LdapUserSearchService(LdapTemplate ldapTemplate) {
        this.ldapTemplate = ldapTemplate;
    }

    // Main method: query validate karun LDAP search karte
    public List<UserSearchDto> searchUsers(String query) {
        String searchValue = validateQuery(query);
        return performLdapSearch(searchValue);
    }

    // Query validate karte
    private String validateQuery(String query) {
        if (query == null || query.trim().isEmpty()) {
            throw new IllegalArgumentException(
                    "Search query is required");
        }

        String searchValue = query.trim();

        if (searchValue.length() > 100) {
            throw new IllegalArgumentException(
                    "Search query must not exceed 100 characters");
        }

        return searchValue;
    }

    // LDAP search filter tayar karte
    private String buildSearchFilter(String searchValue) {
        AndFilter filter = new AndFilter();

        OrFilter userTypeFilter = new OrFilter();
        userTypeFilter.or(
                new EqualsFilter("objectClass", "user"));
        userTypeFilter.or(
                new EqualsFilter("objectClass", "inetOrgPerson"));

        OrFilter searchFilter = new OrFilter();
        searchFilter.or(
                new EqualsFilter("sAMAccountName", searchValue));
        searchFilter.or(
                new WhitespaceWildcardsFilter("cn", searchValue));
        searchFilter.or(
                new WhitespaceWildcardsFilter(
                        "displayName", searchValue));

        filter.and(userTypeFilter);
        filter.and(searchFilter);

        return filter.encode();
    }

    // LDAP madhun matching users retrieve karte
    private List<UserSearchDto> performLdapSearch(
            String searchValue) {

        SearchControls controls = new SearchControls();
        controls.setSearchScope(SearchControls.SUBTREE_SCOPE);
        controls.setCountLimit(100);
        controls.setTimeLimit(5000);

        String filter = buildSearchFilter(searchValue);

        return ldapTemplate.search(
                userSearchBase,
                filter,
                controls,
                (AttributesMapper<UserSearchDto>) this::mapUser
        );
    }

    // LDAP attributes DTO madhye map karte
    private UserSearchDto mapUser(Attributes attributes)
            throws NamingException {

        String adId = getAttribute(
                attributes, "sAMAccountName");

        if (adId == null) {
            adId = getAttribute(attributes, "uid");
        }

        String name = getAttribute(
                attributes, "displayName");

        if (name == null) {
            name = getAttribute(attributes, "cn");
        }

        String emailId = getAttribute(
                attributes, "mail");

        String phoneNumber = getAttribute(
                attributes, "mobile");

        if (phoneNumber == null) {
            phoneNumber = getAttribute(
                    attributes, "telephoneNumber");
        }

        return new UserSearchDto(
                adId,
                name,
                emailId,
                phoneNumber
        );
    }

    // LDAP madhun ek attribute value ghete
    private String getAttribute(
            Attributes attributes,
            String attributeName) throws NamingException {

        if (attributes.get(attributeName) == null) {
            return null;
        }

        Object value = attributes.get(attributeName).get();

        return value == null ? null : value.toString();
    }
}


// ============================================================
// FILE 3: controller/admin/UserSearchController.java
// LoggerUtility / LoggerFactoryUtility che exact imports
// tujhya existing LoginController madhun copy kar.
// ============================================================

package com.epay.admin.portal.controller.admin;

import com.epay.admin.portal.dto.admin.UserSearchDto;
import com.epay.admin.portal.service.admin.LdapUserSearchService;
import io.swagger.v3.oas.annotations.Operation;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

// Existing LoginController madhil logger imports vapra.

@RestController
@RequiredArgsConstructor
@RequestMapping("/admin/users")
public class UserSearchController {

    private final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(this.getClass());

    private final LdapUserSearchService ldapUserSearchService;

    @GetMapping("/search")
    @Operation(
            summary = "Search LDAP Users",
            description = "Search user details by AD ID or name"
    )
    public List<UserSearchDto> searchUsers(
            @RequestParam("query") String query) {

        logger.info("LDAP user search request received");
        return ldapUserSearchService.searchUsers(query);
    }
}


// ============================================================
// FILE 4: LdapConfig.java
// Existing config class madhye LdapTemplate bean nasel tarch
// ha bean add kar. Existing context source vapra.
// ============================================================

@Bean
public LdapTemplate ldapTemplate(
        LdapContextSource contextSource) {
    return new LdapTemplate(contextSource);
}


// ============================================================
// FILE 5: application-local.yml
// Existing auth.ldap block madhye merge kar.
// ============================================================

auth:
  ldap:
    user-search-base: ou=GTMP


// ============================================================
// FILE 6: application-dev.yml
// Existing auth.ldap block madhye merge kar.
// Actual AD user OU verify karun base set kar.
// ============================================================

auth:
  ldap:
    user-search-base: ou=users


// ============================================================
// POSTMAN TEST
// ============================================================

// GET
// http://localhost:8080/admin/users/search?query=V102154
//
// GET
// http://localhost:8080/admin/users/search?query=Rohit
//
// Host, port ani context path tujhya app nusar badal.


// ============================================================
// SAMPLE RESPONSE
// He sample aahe; actual LDAP madhil data parat yeil.
// ============================================================

[
  {
    "adId": "V102154",
    "name": "Rohit Gardi",
    "emailId": "rohit@example.com",
    "phoneNumber": "9876543210"
  }
]






private void validateOtpsByPrefix(List<OtpManagement> otps,
                                  UnblockUserRequest unblockUserRequest) {

    Map<String, OtpManagement> otpMap = otps.stream()
            .collect(Collectors.toMap(
                    otp -> otp.getOtpCode().substring(0, 1),
                    Function.identity()
            ));

    validateSmsOtp(
            otpMap.get(SMS_OTP_PREFIX),
            SMS_OTP_PREFIX + unblockUserRequest.getSmsOtp()
    );

    validateEmailOtp(
            otpMap.get(EMAIL_OTP_PREFIX),
            EMAIL_OTP_PREFIX + unblockUserRequest.getEmailOtp()
    );
}


private void validateSmsOtp(OtpManagement otp, String requestOtp) {

    if (otp == null || !otp.getOtpCode().equals(requestOtp)) {
        throw new MerchantException(
                INVALID_ERROR_CODE,
                MessageFormat.format(
                        INVALID_ERROR_MESSAGE,
                        "SMS OTP",
                        INVALID_OTP_MESSAGE
                )
        );
    }
}


private void validateEmailOtp(OtpManagement otp, String requestOtp) {

    if (otp == null || !otp.getOtpCode().equals(requestOtp)) {
        throw new MerchantException(
                INVALID_ERROR_CODE,
                MessageFormat.format(
                        INVALID_ERROR_MESSAGE,
                        "Email OTP",
                        INVALID_OTP_MESSAGE
                )
        );
    }
}






@@@@@@@@





package com.epay.cs.filter;

import com.epay.cs.constant.CommunicationConstant;
import com.epay.cs.service.ApiRequestResponseLogService;
import com.epay.cs.util.LoggerFactoryUtility;
import com.epay.cs.util.LoggerUtility;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;
import org.springframework.web.util.ContentCachingRequestWrapper;
import org.springframework.web.util.ContentCachingResponseWrapper;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Arrays;
import java.util.Enumeration;
import java.util.UUID;

@Component
@RequiredArgsConstructor
public class ApiRequestResponseLogFilter extends OncePerRequestFilter {

    private final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(this.getClass());

    private final ApiRequestResponseLogService logService;

    @Value("${api.logging.supported-content-types:application/json,text/plain,application/xml,text/xml}")
    private String supportedContentTypes;

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain)
            throws ServletException, IOException {

        String correlationId =
                setDefaultLoggerMDC(request);

        ContentCachingRequestWrapper requestWrapper =
                new ContentCachingRequestWrapper(request);

        ContentCachingResponseWrapper responseWrapper =
                new ContentCachingResponseWrapper(response);

        responseWrapper.setHeader(
                CommunicationConstant.X_CORRELATION_ID,
                correlationId);

        String url =
                request.getRequestURI();

        String method =
                request.getMethod();

        try {

            filterChain.doFilter(
                    requestWrapper,
                    responseWrapper);

        } finally {

            logRequest(
                    requestWrapper,
                    correlationId,
                    method,
                    url);

            logResponse(
                    responseWrapper,
                    correlationId,
                    method,
                    url);

            responseWrapper.copyBodyToResponse();
        }
    }

    private String setDefaultLoggerMDC(
            HttpServletRequest request) {

        String correlationId =
                request.getHeader(
                        CommunicationConstant.X_CORRELATION_ID);

        if (StringUtils.isEmpty(correlationId)) {

            correlationId =
                    UUID.randomUUID().toString();
        }

        LoggerFactoryUtility.putMDC(
                "correlation",
                correlationId);

        LoggerFactoryUtility.putMDC(
                "scenario",
                request.getRequestURI());

        LoggerFactoryUtility.putMDC(
                "operation",
                request.getMethod());

        return correlationId;
    }

    private void logRequest(
            ContentCachingRequestWrapper requestWrapper,
            String correlationId,
            String method,
            String url) {

        String requestContentType =
                requestWrapper.getContentType();

        if (isSupportedContentType(requestContentType)) {

            String requestBody =
                    getRequest(requestWrapper);

            String requestHeaders =
                    getHeaders(requestWrapper);

            logService.buffer(
                    correlationId,
                    CommunicationConstant.REQUEST,
                    method,
                    url,
                    requestBody,
                    requestHeaders);

        } else {

            logger.info(
                    "API request log not saved. " +
                    "Unsupported Content-Type: {}, URL: {}, CorrelationId: {}",
                    requestContentType,
                    url,
                    correlationId);
        }
    }

    private void logResponse(
            ContentCachingResponseWrapper responseWrapper,
            String correlationId,
            String method,
            String url) {

        String responseContentType =
                responseWrapper.getContentType();

        if (isSupportedContentType(responseContentType)) {

            String responseBody =
                    getResponse(responseWrapper);

            String responseHeaders =
                    getHeaders(responseWrapper);

            logService.buffer(
                    correlationId,
                    CommunicationConstant.RESPONSE,
                    method,
                    url,
                    responseBody,
                    responseHeaders);

        } else {

            logger.info(
                    "API response log not saved. " +
                    "Unsupported Content-Type: {}, URL: {}, CorrelationId: {}",
                    responseContentType,
                    url,
                    correlationId);
        }
    }

    private String getRequest(
            ContentCachingRequestWrapper request) {

        byte[] content =
                request.getContentAsByteArray();

        return new String(
                content,
                StandardCharsets.UTF_8);
    }

    private String getResponse(
            ContentCachingResponseWrapper response) {

        byte[] content =
                response.getContentAsByteArray();

        return new String(
                content,
                StandardCharsets.UTF_8);
    }

    private String getHeaders(
            HttpServletRequest request) {

        StringBuilder headers =
                new StringBuilder();

        Enumeration<String> headerNames =
                request.getHeaderNames();

        if (headerNames != null) {

            while (headerNames.hasMoreElements()) {

                String headerName =
                        headerNames.nextElement();

                headers.append(headerName)
                        .append(": ")
                        .append(request.getHeader(headerName))
                        .append("\n");
            }
        }

        return headers.toString();
    }

    private String getHeaders(
            HttpServletResponse response) {

        StringBuilder headers =
                new StringBuilder();

        for (String headerName :
                response.getHeaderNames()) {

            headers.append(headerName)
                    .append(": ")
                    .append(response.getHeader(headerName))
                    .append("\n");
        }

        return headers.toString();
    }

    private boolean isSupportedContentType(
            String contentType) {

        if (contentType == null ||
                contentType.isBlank()) {

            return false;
        }

        return Arrays.stream(
                        supportedContentTypes.split(","))
                .map(String::trim)
                .anyMatch(contentType::startsWith);
    }
}





******
private String setDefaultLoggerMDC(
        HttpServletRequest request) {

    String correlationId =
            request.getHeader(
                    CommunicationConstant.X_CORRELATION_ID);

    if (StringUtils.isEmpty(correlationId)) {
        correlationId =
                UUID.randomUUID().toString();
    }

    LoggerFactoryUtility.putMDC(
            "correlation",
            correlationId);

    LoggerFactoryUtility.putMDC(
            "scenario",
            request.getRequestURI());

    LoggerFactoryUtility.putMDC(
            "operation",
            request.getMethod());

    return correlationId;
}





package com.epay.cs.filter;

import com.epay.cs.constant.CommunicationConstant;
import com.epay.cs.service.ApiRequestResponseLogService;
import com.epay.cs.util.LoggerFactoryUtility;
import com.epay.cs.util.LoggerUtility;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;
import org.springframework.web.util.ContentCachingRequestWrapper;
import org.springframework.web.util.ContentCachingResponseWrapper;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Arrays;
import java.util.Enumeration;
import java.util.UUID;

@Component
@RequiredArgsConstructor
public class ApiRequestResponseLogFilter extends OncePerRequestFilter {

    private final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(this.getClass());

    private final ApiRequestResponseLogService logService;

    @Value("${api.logging.supported-content-types:application/json,text/plain,application/xml,text/xml}")
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
                        CommunicationConstant.X_CORRELATION_ID);

        if (correlationId == null ||
                correlationId.isBlank()) {

            correlationId =
                    UUID.randomUUID().toString();
        }

        response.setHeader(
                CommunicationConstant.X_CORRELATION_ID,
                correlationId);

        String url =
                request.getRequestURI();

        try {

            filterChain.doFilter(
                    requestWrapper,
                    responseWrapper);

        } finally {

            logRequest(
                    requestWrapper,
                    correlationId,
                    url);

            logResponse(
                    responseWrapper,
                    correlationId,
                    url);

            responseWrapper.copyBodyToResponse();
        }
    }

    private void logRequest(
            ContentCachingRequestWrapper requestWrapper,
            String correlationId,
            String url) {

        String requestContentType =
                requestWrapper.getContentType();

        if (isSupportedContentType(requestContentType)) {

            String requestBody =
                    getRequest(requestWrapper);

            String requestHeaders =
                    getHeaders(requestWrapper);

            logService.buffer(
                    correlationId,
                    CommunicationConstant.REQUEST,
                    url,
                    requestBody,
                    requestHeaders);

        } else {

            logger.info(
                    "API request log not saved. " +
                    "Unsupported Content-Type: {}, URL: {}, CorrelationId: {}",
                    requestContentType,
                    url,
                    correlationId);
        }
    }

    private void logResponse(
            ContentCachingResponseWrapper responseWrapper,
            String correlationId,
            String url) {

        String responseContentType =
                responseWrapper.getContentType();

        if (isSupportedContentType(responseContentType)) {

            String responseBody =
                    getResponse(responseWrapper);

            String responseHeaders =
                    getHeaders(responseWrapper);

            logService.buffer(
                    correlationId,
                    CommunicationConstant.RESPONSE,
                    url,
                    responseBody,
                    responseHeaders);

        } else {

            logger.info(
                    "API response log not saved. " +
                    "Unsupported Content-Type: {}, URL: {}, CorrelationId: {}",
                    responseContentType,
                    url,
                    correlationId);
        }
    }

    private String getRequest(
            ContentCachingRequestWrapper request) {

        byte[] content =
                request.getContentAsByteArray();

        return new String(
                content,
                StandardCharsets.UTF_8);
    }

    private String getResponse(
            ContentCachingResponseWrapper response) {

        byte[] content =
                response.getContentAsByteArray();

        return new String(
                content,
                StandardCharsets.UTF_8);
    }

    private String getHeaders(
            HttpServletRequest request) {

        StringBuilder headers =
                new StringBuilder();

        Enumeration<String> headerNames =
                request.getHeaderNames();

        if (headerNames != null) {

            while (headerNames.hasMoreElements()) {

                String headerName =
                        headerNames.nextElement();

                headers.append(headerName)
                        .append(": ")
                        .append(request.getHeader(headerName))
                        .append("\n");
            }
        }

        return headers.toString();
    }

    private String getHeaders(
            HttpServletResponse response) {

        StringBuilder headers =
                new StringBuilder();

        for (String headerName :
                response.getHeaderNames()) {

            headers.append(headerName)
                    .append(": ")
                    .append(response.getHeader(headerName))
                    .append("\n");
        }

        return headers.toString();
    }

    private boolean isSupportedContentType(
            String contentType) {

        if (contentType == null ||
                contentType.isBlank()) {

            return false;
        }

        return Arrays.stream(
                        supportedContentTypes.split(","))
                .map(String::trim)
                .anyMatch(contentType::startsWith);
    }
}




****
package com.epay.cs.filter;

import com.epay.cs.constant.CommunicationConstant;
import com.epay.cs.service.ApiRequestResponseLogService;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;
import org.springframework.web.util.ContentCachingRequestWrapper;
import org.springframework.web.util.ContentCachingResponseWrapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Arrays;
import java.util.Enumeration;
import java.util.UUID;

@Component
@RequiredArgsConstructor
public class ApiRequestResponseLogFilter
        extends OncePerRequestFilter {

    private final Logger logger =
            LoggerFactory.getLogger(this.getClass());

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
                        CommunicationConstant.X_CORRELATION_ID
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

                String requestHeaders =
                        getHeaders(requestWrapper);

                logService.buffer(
                        correlationId,
                        CommunicationConstant.REQUEST,
                        url,
                        requestBody,
                        requestHeaders
                );

            } else {

                logger.info(
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

                String responseHeaders =
                        getHeaders(responseWrapper);

                logService.buffer(
                        correlationId,
                        CommunicationConstant.RESPONSE,
                        url,
                        responseBody,
                        responseHeaders
                );

            } else {

                logger.info(
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

    private String getHeaders(
            HttpServletRequest request) {

        StringBuilder headers =
                new StringBuilder();

        Enumeration<String> headerNames =
                request.getHeaderNames();

        if (headerNames != null) {

            while (headerNames.hasMoreElements()) {

                String headerName =
                        headerNames.nextElement();

                headers.append(headerName)
                        .append(": ")
                        .append(request.getHeader(headerName))
                        .append("\n");
            }
        }

        return headers.toString();
    }

    private String getHeaders(
            HttpServletResponse response) {

        StringBuilder headers =
                new StringBuilder();

        for (String headerName :
                response.getHeaderNames()) {

            headers.append(headerName)
                    .append(": ")
                    .append(response.getHeader(headerName))
                    .append("\n");
        }

        return headers.toString();
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






****"



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
