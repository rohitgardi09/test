private static final AttributesMapper<UserSearchDto> USER_MAPPER = attrs -> {

    String adId = attrs.get("sAMAccountName") != null
            ? attrs.get("sAMAccountName").get().toString()
            : null;

    String firstName = attrs.get("givenName") != null
            ? attrs.get("givenName").get().toString()
            : null;

    String lastName = null;
    if (attrs.get("cn") != null) {
        String fullName = attrs.get("cn").get().toString().trim();
        String[] parts = fullName.split("\\s+");
        lastName = parts.length > 1
                ? parts[parts.length - 1]
                : null;
    }

    String email = attrs.get("mail") != null
            ? attrs.get("mail").get().toString()
            : null;

    return new UserSearchDto(adId, firstName, lastName, email);
};
