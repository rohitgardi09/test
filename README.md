OrFilter objectClassFilter = new OrFilter();
objectClassFilter.or(new EqualsFilter("objectClass", "user"));
objectClassFilter.or(new EqualsFilter("objectClass", "inetOrgPerson"));

OrFilter matchFilter = new OrFilter();
matchFilter.or(new EqualsFilter("sAMAccountName", userId));
matchFilter.or(new LikeFilter("cn", "*" + userId + "*"));

AndFilter filter = new AndFilter();
filter.and(objectClassFilter);
filter.and(matchFilter);

List<UserSearchDto> results =
        ldapTemplate.search(userSearchBase, filter.encode(), USER_MAPPER);

return results;
