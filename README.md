public List<UserSearchDto> search(String userId) {

    log.info("LDAP user search started.");

    try {
        if (userId == null || userId.trim().isEmpty()) {
            log.info("LDAP user search skipped because userId is empty.");
            return Collections.emptyList();
        }

        String searchValue = userId.trim();

        // First check ADID in User MView
        UserMViewDto userMViewDto =
                userMViewDao.getUserDetails(
                        searchValue.toUpperCase(), UserStatus.ACTIVE);

        // If ADID found in View, return View details
        if (userMViewDto != null) {

            log.info("User found in User MView: {}", searchValue);

            UserSearchDto userSearchDto = new UserSearchDto(
                    userMViewDto.getAdId(),
                    userMViewDto.getFirstName(),
                    userMViewDto.getLastName(),
                    userMViewDto.getEmail()
            );

            return Collections.singletonList(userSearchDto);
        }

        log.info("User not found in View. Searching LDAP by ADID: {}", searchValue);

        OrFilter objectClassFilter = new OrFilter();
        objectClassFilter.or(new EqualsFilter("objectClass", "user"));
        objectClassFilter.or(new EqualsFilter("objectClass", "inetOrgPerson"));

        OrFilter matchFilter = new OrFilter();
        matchFilter.or(new EqualsFilter("sAMAccountName", searchValue));
        matchFilter.or(new LikeFilter("cn", "*" + searchValue + "*"));

        AndFilter filter = new AndFilter();
        filter.and(objectClassFilter);
        filter.and(matchFilter);

        log.info("LDAP search filter created successfully.");

        List<UserSearchDto> results =
                ldapTemplate.search(
                        userSearchBase,
                        filter.encode(),
                        USER_MAPPER);

        log.info("LDAP user search completed successfully. Result count: {}", results.size());

        return results;

    } catch (Exception ex) {

        log.error("LDAP user search failed. error: {}", ex.getMessage());

        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                MessageFormat.format(
                        FAILED_ERROR_MESSAGE_WITH_REASON,
                        LDAP_USER_SEARCH,
                        ex.getMessage()));
    }
}
