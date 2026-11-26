
public void validateAllowedSecondaryRole(
        String requestedSecondaryRole,
        List<String> allowedSecondaryRoleCodes) {

    // Check requested role and allowed role list
    if (requestedSecondaryRole == null
            || requestedSecondaryRole.isBlank()
            || allowedSecondaryRoleCodes == null
            || allowedSecondaryRoleCodes.isEmpty()) {

        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SECONDARY_ROLE_NOT_ALLOWED
        );
    }

    // Check whether requested role is present in allowed list
    boolean isAllowed = allowedSecondaryRoleCodes.stream()
            .filter(Objects::nonNull)
            .map(String::trim)
            .anyMatch(code ->
                    code.equalsIgnoreCase(requestedSecondaryRole.trim())
            );

    // Throw exception if role is not allowed
    if (!isAllowed) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SECONDARY_ROLE_NOT_ALLOWED
        );
    }
}





List<String> allowedSecondaryRoleCodes =
        primaryBusinessRole.getSecondaryBrCode();

userManagementValidator.validateAllowedSecondaryRole(
        request.getSecondaryRole(),
        allowedSecondaryRoleCodes
);
