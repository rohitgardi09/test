// Imports to add in LdapAuthenticationService.java
import com.epay.admin.portal.model.request.LoginRequest;
import org.springframework.ldap.CommunicationException;
import org.springframework.ldap.core.DirContextOperations;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import javax.naming.directory.DirContext;

public String testServiceAccountBind(LoginRequest loginRequest) {

    StringBuilder result = new StringBuilder();
    String msg;

    // STAGE 0: Raw TCP connectivity check
    String host;
    int port;
    try {
        java.net.URI uri = java.net.URI.create(contextSource.getUrls()[0]);
        host = uri.getHost();
        port = uri.getPort() != -1 ? uri.getPort() : ("ldaps".equalsIgnoreCase(uri.getScheme()) ? 636 : 389);
    } catch (Exception ex) {
        msg = "STAGE 0 FAILED - CONFIG ISSUE: cannot parse ldap url. Msg: " + ex.getMessage();
        log.error(msg, ex);
        return result.append(msg).toString();
    }
    try (java.net.Socket socket = new java.net.Socket()) {
        socket.connect(new java.net.InetSocketAddress(host, port), 5000);
        msg = "STAGE 0 RESULT: SUCCESS - TCP connection to " + host + ":" + port + " is open";
        log.info(msg);
        result.append(msg).append("\n");
    } catch (java.net.SocketTimeoutException ex) {
        msg = "STAGE 0 FAILED - NETWORK ISSUE: TIMEOUT connecting to " + host + ":" + port + " - firewall/VPN likely blocking this port. Not a config/code issue.";
        log.error(msg, ex);
        return result.append(msg).toString();
    } catch (java.io.IOException ex) {
        msg = "STAGE 0 FAILED - NETWORK ISSUE: Cannot connect to " + host + ":" + port + " - Type: " + ex.getClass().getName() + ", Msg: " + ex.getMessage();
        log.error(msg, ex);
        return result.append(msg).toString();
    }

    // STAGE 1: Effective config
    log.info("Configured URL: {}", String.join(",", contextSource.getUrls()));
    log.info("Configured Base DN: {}", contextSource.getBaseLdapPathAsString());
    log.info("Service Account UserDn: {}", contextSource.getUserDn());
    result.append("URL: ").append(String.join(",", contextSource.getUrls())).append("\n")
          .append("Base DN: ").append(contextSource.getBaseLdapPathAsString()).append("\n")
          .append("Service Account UserDn: ").append(contextSource.getUserDn()).append("\n")
          .append("Service Account password length: ").append(contextSource.getPassword() != null ? contextSource.getPassword().length() : "NULL").append("\n");
    try {
        contextSource.afterPropertiesSet();
        msg = "STAGE 1 RESULT: contextSource initialized successfully";
        log.info(msg);
        result.append(msg).append("\n");
    } catch (Exception ex) {
        msg = "STAGE 1 FAILED - CONFIG ISSUE: ldap.* properties malformed. Type: " + ex.getClass().getName() + ", Msg: " + ex.getMessage();
        log.error(msg, ex);
        return result.append(msg).toString();
    }

    // STAGE 2: Service account bind
    try {
        contextSource.getContext(contextSource.getUserDn(), contextSource.getPassword());
        msg = "STAGE 2 RESULT: Service account bind SUCCESS - AD server reachable, service account credentials correct";
        log.info(msg);
        result.append(msg).append("\n");
    } catch (CommunicationException ex) {
        msg = "STAGE 2 FAILED - CONNECTION/SSL ISSUE: reached TCP but LDAPS handshake failed. Check cert trust. Msg: " + ex.getMessage();
        log.error(msg, ex);
        return result.append(msg).toString();
    } catch (org.springframework.ldap.AuthenticationException ex) {
        String err = String.valueOf(ex.getMessage());
        String reason = err.contains("data 52e") ? "WRONG PASSWORD (or LDAP_SERVICE_PASSWORD not loaded)"
                : err.contains("data 525") ? "USER DN NOT FOUND - check spring.ldap.username DN"
                : err.contains("data 532") ? "PASSWORD EXPIRED"
                : err.contains("data 533") ? "ACCOUNT DISABLED"
                : err.contains("data 775") ? "ACCOUNT LOCKED"
                : err.contains("data 701") ? "ACCOUNT EXPIRED"
                : "UNKNOWN AD CODE - see message";
        msg = "STAGE 2 FAILED - " + reason + " | Msg: " + err;
        log.error(msg, ex);
        return result.append(msg).toString();
    } catch (Exception ex) {
        msg = "STAGE 2 FAILED - UNKNOWN: Type: " + ex.getClass().getName() + ", Msg: " + ex.getMessage();
        log.error(msg, ex);
        return result.append(msg).toString();
    }

    // STAGE 3: User search
    FilterBasedLdapUserSearch ldapUserSearch = new FilterBasedLdapUserSearch(userSearchBase, userSearchFilter, contextSource);
    ldapUserSearch.setSearchSubtree(true);
    DirContextOperations userDetails;
    try {
        userDetails = ldapUserSearch.searchForUser(loginRequest.getUserId());
        msg = "STAGE 3 RESULT: User found. Resolved DN: " + userDetails.getDn();
        log.info(msg);
        result.append(msg).append("\n");
    } catch (UsernameNotFoundException ex) {
        msg = "STAGE 3 FAILED - CONFIG ISSUE: user '" + loginRequest.getUserId() + "' not found. Check user-search-base/user-search-filter. Msg: " + ex.getMessage();
        log.error(msg, ex);
        return result.append(msg).toString();
    } catch (Exception ex) {
        msg = "STAGE 3 FAILED - UNKNOWN: Type: " + ex.getClass().getName() + ", Msg: " + ex.getMessage();
        log.error(msg, ex);
        return result.append(msg).toString();
    }

    // STAGE 4: User bind
    DirContext context = null;
    try {
        context = contextSource.getContext(userDetails.getDn().toString(), loginRequest.getPassword());
        msg = "STAGE 4 RESULT: SUCCESS - user authenticated correctly. No issue anywhere.";
        log.info(msg);
        result.append(msg);
    } catch (org.springframework.ldap.AuthenticationException ex) {
        msg = "STAGE 4 FAILED - CREDENTIAL ISSUE: DN resolved correctly, password wrong or account locked/disabled. Msg: " + ex.getMessage();
        log.error(msg, ex);
        result.append(msg);
    } catch (CommunicationException ex) {
        msg = "STAGE 4 FAILED - CONNECTION DROPPED mid-flow. Msg: " + ex.getMessage();
        log.error(msg, ex);
        result.append(msg);
    } catch (Exception ex) {
        msg = "STAGE 4 FAILED - UNKNOWN: Type: " + ex.getClass().getName() + ", Msg: " + ex.getMessage();
        log.error(msg, ex);
        result.append(msg);
    } finally {
        if (context != null) {
            try {
                context.close();
            } catch (Exception ex) {
                log.info("Failed to close LDAP context", ex);
            }
        }
    }
    return result.toString();
}
