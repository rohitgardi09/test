public List<UserSearchDto> search(String userId) {

    log.info("LDAP user search started.");

    try {
        if (userId == null || userId.trim().isEmpty()) {
            log.info("LDAP user search skipped because userId is empty.");
            return Collections.emptyList();
        }

        String searchValue = userId.trim();

        OrFilter objectClassFilter = new OrFilter();
        objectClassFilter.or(new EqualsFilter("objectClass", "user"));
        objectClassFilter.or(new EqualsFilter("objectClass", "inetOrgPerson"));

        OrFilter matchFilter = new OrFilter();
        matchFilter.or(
                new EqualsFilter("sAMAccountName", searchValue)
        );
        matchFilter.or(
                new LikeFilter("cn", "*" + searchValue + "*")
        );

        AndFilter filter = new AndFilter();
        filter.and(objectClassFilter);
        filter.and(matchFilter);

        log.info("LDAP search filter created successfully.");

        List<UserSearchDto> results = ldapTemplate.search(
                userSearchBase,
                filter.encode(),
                USER_MAPPER
        );

        log.info(
                "LDAP user search completed successfully. Result count: {}",
                results.size()
        );

        return results;

    } catch (Exception e) {
        log.error("LDAP user search failed.", e);
        throw e;
    }
}
