public void authenticate(LoginRequest loginRequest) {
    try {
        log.info("Starting LDAP service account authentication");
        contextSource.afterPropertiesSet();

        contextSource.getContext(contextSource.getUserDn(), contextSource.getPassword());
        log.info("LDAP service account authentication completed successfully");

        FilterBasedLdapUserSearch userSearch = new FilterBasedLdapUserSearch(userSearchBase, userSearchFilter, contextSource);
        userSearch.setSearchSubtree(true);

        log.info("Searching LDAP user: {}", loginRequest.getUserId());
        DirContextOperations userDetails = userSearch.searchForUser(loginRequest.getUserId());
        log.info("LDAP user found successfully");

        log.info("Validating LDAP user credentials");
        contextSource.getContext(userDetails.getNameInNamespace(), loginRequest.getPassword());
        log.info("LDAP user authentication completed successfully");

    } catch (LockedException ex) {
        log.error("LDAP user account is locked. UserId: {}, error: {}", loginRequest.getUserId(), ex.getMessage());
        throw new AdminPortalException(AD_ID_BLOCKED_ERROR_CODE, AD_ID_BLOCKED_ERROR_MESSAGE);

    } catch (BadCredentialsException ex) {
        log.error("LDAP authentication failed due to invalid credentials. UserId: {}, error: {}", loginRequest.getUserId(), ex.getMessage());
        throw new AdminPortalException(INCORRECT_AD_ID_ERROR_CODE, INCORRECT_AD_ID_ERROR_MESSAGE);

    } catch (Exception ex) {
        log.error("LDAP authentication failed. UserId: {}, error: {}", loginRequest.getUserId(), ex.getMessage());
        throw new AdminPortalException(INCORRECT_AD_ID_ERROR_CODE, INCORRECT_AD_ID_ERROR_MESSAGE);
    }
}
