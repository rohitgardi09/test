// ========================================================================
// FILE 1: src/main/java/com/epay/admin/portal/dto/admin/UserSearchDto.java
// ========================================================================
package com.epay.admin.portal.dto.admin;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserSearchDto {
    private String adId;
    private String firstName;
    private String lastName;
    private String emailId;
}


// ========================================================================
// FILE 2: src/main/java/com/epay/admin/portal/service/LdapAuthenticationService.java
// ADD these imports, this field, and this method to the EXISTING file.
// Do not remove anything already present (authenticate(), testServiceAccountBind()).
// ========================================================================

// --- Add these imports alongside existing imports ---
import com.epay.admin.portal.dto.admin.UserSearchDto;
import org.springframework.ldap.core.AttributesMapper;
import org.springframework.ldap.core.LdapTemplate;
import org.springframework.ldap.filter.AndFilter;
import org.springframework.ldap.filter.EqualsFilter;
import org.springframework.ldap.filter.LikeFilter;
import org.springframework.ldap.filter.OrFilter;
import javax.naming.directory.Attributes;
import java.util.List;

// --- Add this field alongside the existing contextSource field ---
private final LdapTemplate ldapTemplate;

// --- Add this mapper + method anywhere inside the class ---
private static final AttributesMapper<UserSearchDto> USER_MAPPER = (Attributes attrs) -> {
    String adId = attrs.get("sAMAccountName") != null ? (String) attrs.get("sAMAccountName").get() : null;
    String fullName = attrs.get("cn") != null ? (String) attrs.get("cn").get() : null;
    String email = attrs.get("mail") != null ? (String) attrs.get("mail").get() : null;

    String firstName = null;
    String lastName = null;
    if (fullName != null) {
        String[] parts = fullName.trim().split("\\s+", 2);
        firstName = parts[0];
        lastName = parts.length > 1 ? parts[1] : null;
    }

    return new UserSearchDto(adId, firstName, lastName, email);
};

/**
 * Searches LDAP for users matching the given query, by ADID (exact match) or name (partial match).
 *
 * @param query ADID or name to search for
 * @return list of matching users with ADID, first name, last name and email
 */
public List<UserSearchDto> search(String query) {
    log.info("Searching LDAP for users matching query: {}", query);

    OrFilter match = new OrFilter();
    match.or(new EqualsFilter("sAMAccountName", query));
    match.or(new LikeFilter("cn", "*" + query + "*"));

    AndFilter filter = new AndFilter();
    filter.and(new EqualsFilter("objectClass", "user"));
    filter.and(match);

    log.debug("LDAP search base: {}, filter: {}", userSearchBase, filter.encode());

    List<UserSearchDto> results = ldapTemplate.search(userSearchBase, filter.encode(), USER_MAPPER);

    log.info("LDAP search completed for query: {}. Found {} user(s)", query, results.size());
    return results;
}


// ========================================================================
// FILE 3: src/main/java/com/epay/admin/portal/controller/LdapUserSearchController.java
// ========================================================================
package com.epay.admin.portal.controller;

import com.epay.admin.portal.dto.admin.UserSearchDto;
import com.epay.admin.portal.service.LdapAuthenticationService;
import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/ldap/users")
@RequiredArgsConstructor
public class LdapUserSearchController {

    private final LoggerUtility log = LoggerFactoryUtility.getLogger(this.getClass());

    private final LdapAuthenticationService ldapAuthenticationService;

    @GetMapping("/search")
    public List<UserSearchDto> search(@RequestParam("query") String query) {
        log.info("Received request to search LDAP users with query: {}", query);
        return ldapAuthenticationService.search(query.trim());
    }
}
