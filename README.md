============================================================
1) FILE: dto/admin/UserSearchDto.java
============================================================

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


============================================================
2) FILE: service/admin/LdapUserSearchService.java
============================================================

package com.epay.admin.portal.service.admin;

import com.epay.admin.portal.dto.admin.UserSearchDto;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.ldap.core.AttributesMapper;
import org.springframework.ldap.core.LdapTemplate;
import org.springframework.ldap.filter.OrFilter;
import org.springframework.ldap.filter.EqualsFilter;
import org.springframework.ldap.filter.WhitespaceWildcardsFilter;
import org.springframework.stereotype.Service;

import javax.naming.NamingException;
import javax.naming.directory.Attributes;
import java.util.List;

@Service
public class LdapUserSearchService {

    private final LdapTemplate ldapTemplate;

    @Value("${auth.ldap.user-search-base}")
    private String userSearchBase;

    public LdapUserSearchService(LdapTemplate ldapTemplate) {
        this.ldapTemplate = ldapTemplate;
    }

    // Query validate karun LDAP search karte
    public List<UserSearchDto> searchUsers(String query) {
        String searchValue = validateQuery(query);

        return ldapTemplate.search(
                userSearchBase,
                buildSearchFilter(searchValue),
                (AttributesMapper<UserSearchDto>) this::mapUser
        );
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

    // AD ID exact kiwa name partial match filter
    private String buildSearchFilter(String searchValue) {
        OrFilter filter = new OrFilter();

        filter.or(new EqualsFilter(
                "sAMAccountName", searchValue));

        filter.or(new WhitespaceWildcardsFilter(
                "cn", searchValue));

        filter.or(new WhitespaceWildcardsFilter(
                "displayName", searchValue));

        return filter.encode();
    }

    // LDAP madhil user attributes DTO madhye map karte
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

    // LDAP madhun attribute value ghete
    private String getAttribute(
            Attributes attributes,
            String attributeName) throws NamingException {

        if (attributes.get(attributeName) == null) {
            return null;
        }









============================================================
FILE 1: dto/admin/UserSearchDto.java
============================================================

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


============================================================
FILE 2: service/admin/LdapUserSearchService.java
============================================================

package com.epay.admin.portal.service.admin;

import com.epay.admin.portal.dto.admin.UserSearchDto;
import com.epay.admin.portal.model.response.AdminPortalResponse;
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

    // Main method: validate, search ani response tayar karte
    public AdminPortalResponse<UserSearchDto> searchUsers(String query) {

        String searchValue = validateQuery(query);

        List<UserSearchDto> users = performLdapSearch(searchValue);

        return buildResponse(users);
    }

    // Search query validate karte
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

    // AD ID kiwa name nusar LDAP filter tayar karte
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

    // LDAP madhun users shodhte
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

    // Project chya AdminPortalResponse format madhye response tayar karte
    private AdminPortalResponse<UserSearchDto> buildResponse(
            List<UserSearchDto> users) {

        long resultCount = users.size();

        return AdminPortalResponse.<UserSearchDto>builder()
                .status(200)
                .data(users)
                .count(resultCount)
                .total(resultCount)
                .build();
    }
}


============================================================
FILE 3: EXISTING controller/admin/LoginController.java
============================================================

Tujhya existing LoginController madhye navin class banvu nakos.
Existing imports madhye he add kar, jar already nasel tar:

import com.epay.admin.portal.dto.admin.UserSearchDto;
import com.epay.admin.portal.service.admin.LdapUserSearchService;
import com.epay.admin.portal.model.response.AdminPortalResponse;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import java.util.List;

Existing LoginController madhye, loginService field chya khali
ha field add kar:

private final LdapUserSearchService ldapUserSearchService;

Existing class madhye refreshToken() method nantar ani
class chya shevatchya } chya aadhi ha endpoint add kar:

@GetMapping("/users/search")
@Operation(
        summary = "Search LDAP Users",
        description = "Search user details by AD ID or name"
)
public AdminPortalResponse<UserSearchDto> searchUsers(
        @RequestParam("query") String query) {

    logger.info("LDAP user search request: {}", query);
    return ldapUserSearchService.searchUsers(query);
}


============================================================
FILE 4: LdapConfig.java
============================================================

Existing LdapConfig madhye LdapTemplate bean nasel tarch
ha bean add kar. Existing selected LdapContextSource vapra.

Imports (jar already nasel tar):

import org.springframework.context.annotation.Bean;
import org.springframework.ldap.core.LdapTemplate;
import org.springframework.ldap.core.support.LdapContextSource;

Bean, existing LdapConfig class chya aat:

@Bean
public LdapTemplate ldapTemplate(
        LdapContextSource contextSource) {

    return new LdapTemplate(contextSource);
}


============================================================
FILE 5: application-local.yml
============================================================

Existing auth.ldap block madhye hi property merge kar.
Duplicate auth: block tayar karu nakos.

auth:
  ldap:
    user-search-base: ou=GTMP


============================================================
FILE 6: application-dev.yml
============================================================

Existing auth.ldap block madhye hi property merge kar.
Actual AD user OU verify karun search base set kar.

auth:
  ldap:
    user-search-base: ou=users


============================================================
POSTMAN REQUEST
============================================================

Method: GET

URL example:
http://localhost:8080/login/users/search?query=V102154

Name ne search:
http://localhost:8080/login/users/search?query=Rohit


============================================================
SAMPLE RESPONSE
============================================================

{
  "status": 200,
  "data": [
    {
      "adId": "V102154",
      "name": "Rohit Gardi",
      "emailId": "rohit@example.com",
      "phoneNumber": "9876543210"
    }
  ],
  "count": 1,
  "total": 1
}
