
public void validateAllowedSecondaryRole(
        String requestedSecondaryRole,
        List<String> secondaryBrCode) {

    if (requestedSecondaryRole == null
            || requestedSecondaryRole.isBlank()
            || secondaryBrCode == null
            || secondaryBrCode.isEmpty()) {

        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SECONDARY_ROLE_NOT_ALLOWED
        );
    }

    // Convert comma-separated values into List
    List<String> allowedRoles = secondaryBrCode.stream()
            .filter(Objects::nonNull)
            .flatMap(value -> Arrays.stream(value.split(",")))
            .map(String::trim)
            .filter(role -> !role.isEmpty())
            .toList();

    // Validate requested Secondary Role
    boolean isAllowed = allowedRoles.stream()
            .anyMatch(role ->
                    role.equalsIgnoreCase(requestedSecondaryRole.trim())
            );

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

