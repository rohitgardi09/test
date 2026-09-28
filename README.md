// Imports to add in LdapAuthenticationService.java
import com.epay.admin.portal.model.request.LoginRequest;
import org.springframework.ldap.CommunicationException;
import org.springframework.ldap.core.DirContextOperations;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import javax.naming.directory.DirContext;

public void testServiceAccountBind(LoginRequest loginRequest) {

    // STAGE 0: Raw TCP connectivity check (host/port taken from configured ldap url)
    String host;
    int port;
    try {
        java.net.URI uri = java.net.URI.create(contextSource.getUrls()[0]);
        host = uri.getHost();
        port = uri.getPort() != -1 ? uri.getPort() : ("ldaps".equalsIgnoreCase(uri.getScheme()) ? 636 : 389);
    } catch (Exception ex) {
        log.error("STAGE 0 FAILED - CONFIG ISSUE: cannot parse ldap url. Msg: {}", ex.getMessage(), ex);
        return;
    }
    try (java.net.Socket socket = new java.net.Socket()) {
        socket.connect(new java.net.InetSocketAddress(host, port), 5000);
        log.info("STAGE 0 RESULT: SUCCESS - TCP connection to {}:{} is open", host, port);
    } catch (java.net.SocketTimeoutException ex) {
        log.error("STAGE 0 FAILED - NETWORK ISSUE: TIMEOUT connecting to {}:{} - firewall/VPN likely blocking this port. Not a config/code issue.", host, port, ex);
        return;
    } catch (java.io.IOException ex) {
        log.error("STAGE 0 FAILED - NETWORK ISSUE: Cannot connect to {}:{} - Type: {}, Msg: {}", host, port, ex.getClass().getName(), ex.getMessage(), ex);
        return;
    }

    // STAGE 1: Log effective config being used
    log.info("Configured URL: {}", String.join(",", contextSource.getUrls()));
    log.info("Configured Base DN: {}", contextSource.getBaseLdapPathAsString());
    log.info("Service Account UserDn: {}", contextSource.getUserDn());
    try {
        contextSource.afterPropertiesSet();
        log.info("STAGE 1 RESULT: contextSource initialized successfully");
    } catch (Exception ex) {
        log.error("STAGE 1 FAILED - CONFIG ISSUE: application.yml ldap.* properties malformed. Type: {}, Msg: {}",
                ex.getClass().getName(), ex.getMessage(), ex);
        return;
    }

    // STAGE 2: Service account bind (proves connection + service account creds work)
    try {
        contextSource.getContext(contextSource.getUserDn(), contextSource.getPassword());
        log.info("STAGE 2 RESULT: Service account bind SUCCESS - AD server reachable, service account credentials correct");
    } catch (CommunicationException ex) {
        log.error("STAGE 2 FAILED - CONNECTION/SSL ISSUE: reached TCP but LDAPS handshake failed. Check cert trust. Msg: {}", ex.getMessage(), ex);
        return;
    } catch (org.springframework.ldap.AuthenticationException ex) {
        log.error("STAGE 2 FAILED - CONFIG/CREDENTIAL ISSUE: service account username/password is wrong. Msg: {}", ex.getMessage(), ex);
        return;
    } catch (Exception ex) {
        log.error("STAGE 2 FAILED - UNKNOWN: Type: {}, Msg: {}", ex.getClass().getName(), ex.getMessage(), ex);
        return;
    }

    // STAGE 3: Search for the user via configured search-base/search-filter
    FilterBasedLdapUserSearch ldapUserSearch = new FilterBasedLdapUserSearch(userSearchBase, userSearchFilter, contextSource);
    ldapUserSearch.setSearchSubtree(true);
    DirContextOperations userDetails;
    try {
        userDetails = ldapUserSearch.searchForUser(loginRequest.getUserId());
        log.info("STAGE 3 RESULT: User found. Resolved DN: {}", userDetails.getDn());
    } catch (UsernameNotFoundException ex) {
        log.error("STAGE 3 FAILED - CONFIG ISSUE: user '{}' not found. Check user-search-base/user-search-filter or confirm user exists under that OU. Msg: {}",
                loginRequest.getUserId(), ex.getMessage(), ex);
        return;
    } catch (Exception ex) {
        log.error("STAGE 3 FAILED - UNKNOWN: Type: {}, Msg: {}", ex.getClass().getName(), ex.getMessage(), ex);
        return;
    }

    // STAGE 4: Actual user bind with their password
    DirContext context = null;
    try {
        context = contextSource.getContext(userDetails.getDn().toString(), loginRequest.getPassword());
        log.info("STAGE 4 RESULT: SUCCESS - user authenticated correctly. No issue anywhere.");
    } catch (org.springframework.ldap.AuthenticationException ex) {
        log.error("STAGE 4 FAILED - CREDENTIAL ISSUE: DN resolved correctly, password wrong or account locked/disabled. Msg: {}", ex.getMessage(), ex);
    } catch (CommunicationException ex) {
        log.error("STAGE 4 FAILED - CONNECTION DROPPED mid-flow - intermittent network issue. Msg: {}", ex.getMessage(), ex);
    } catch (Exception ex) {
        log.error("STAGE 4 FAILED - UNKNOWN: Type: {}, Msg: {}", ex.getClass().getName(), ex.getMessage(), ex);
    } finally {
        if (context != null) {
            try {
                context.close();
            } catch (Exception ex) {
                log.info("Failed to close LDAP context", ex);
            }
        }
    }
}
