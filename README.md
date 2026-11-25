
public void validateAllowedSecondaryRole(
        String requestedSecondaryRole,
        String allowedSecondaryRoleCodes) {

    if (requestedSecondaryRole == null
            || requestedSecondaryRole.isBlank()
            || allowedSecondaryRoleCodes == null
            || allowedSecondaryRoleCodes.isBlank()) {

        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SECONDARY_ROLE_NOT_ALLOWED
        );
    }

    try {
        List<String> allowedRoles = objectMapper.readValue(
                allowedSecondaryRoleCodes,
                new TypeReference<List<String>>() {}
        );

        boolean isAllowed = allowedRoles.stream()
                .anyMatch(role -> role.equalsIgnoreCase(
                        requestedSecondaryRole.trim()
                ));

        if (!isAllowed) {
            throw new AdminPortalException(
                    FAILED_ERROR_CODE,
                    SECONDARY_ROLE_NOT_ALLOWED
            );
        }

    } catch (JsonProcessingException e) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SECONDARY_ROLE_NOT_ALLOWED
        );
    }
}




m?mmmmmmmmm
private void updateSecondaryRole(UpdateUserRequest updateUserRequest) {
    try {
        log.info("Starting Secondary Role Assignment");

        List<SecondaryRoleAssignmentRequest> assignments =
                updateUserRequest.getSecondaryRoleAssignment();

        userManagementValidator.validateSecondaryRoleAssignments(assignments);

        for (SecondaryRoleAssignmentRequest request : assignments) {

            UUID primaryUserId =
                    userManagementValidator.getUserId(request.getPrimaryAdId());

            UUID secondaryUserId =
                    userManagementValidator.getUserId(request.getSecondaryAdId());

            UserRole primaryRole =
                    userManagementValidator.getPrimaryRole(primaryUserId);

            // Fetch Primary Business Role
            BusinessRolesMaster primaryBusinessRole =
                    businessRolesMasterRepository.findById(primaryRole.getRoleId())
                            .orElseThrow(() -> new AdminPortalException(
                                    GENERIC_ERROR_CODE,
                                    "Primary Business Role not found"
                            ));

            // Validate allowed Secondary Role from JSON array
            userManagementValidator.validateAllowedSecondaryRole(
                    request.getSecondaryRole(),
                    primaryBusinessRole.getSecondaryBrCodes()
            );

            // Fetch selected Secondary Role
            BusinessRolesMaster secondaryRole =
                    userManagementValidator.getSecondaryRole(
                            request.getSecondaryRole()
                    );

            userManagementValidator.validateDuplicateSecondaryRole(
                    primaryUserId,
                    secondaryUserId,
                    secondaryRole.getId()
            );

            saveSecondaryUserRole(
                    primaryUserId,
                    secondaryUserId,
                    primaryRole,
                    secondaryRole,
                    request
            );
        }

        log.info("Secondary Role Assignment completed successfully");

    } catch (Exception e) {
        log.error("Error while assigning Secondary Role", e);
        throw new AdminPortalException(GENERIC_ERROR_CODE, GENERIC_ERROR_MESSAGE);
    }
}
