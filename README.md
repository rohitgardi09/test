
/* =========================================================
   1. UserManagementDao.java
   ========================================================= */

private void updateSecondaryRole(UpdateUserRequest updateUserRequest) {
    try {
        log.info("Starting Secondary Role Assignment");

        List<SecondaryRoleAssignmentRequest> assignments =
                updateUserRequest.getSecondaryRoleAssignment();

        userManagementValidator.validateSecondaryRoleAssignments(assignments);

        for (SecondaryRoleAssignmentRequest request : assignments) {

            // Fetch Primary User
            UUID primaryUserId =
                    userManagementValidator.getUserId(request.getPrimaryAdId());

            // Fetch Secondary User
            UUID secondaryUserId =
                    userManagementValidator.getUserId(request.getSecondaryAdId());

            // Fetch Primary Role
            UserRole primaryRole =
                    userManagementValidator.getPrimaryRole(primaryUserId);

            // Fetch Primary Business Role
            BusinessRolesMaster primaryBusinessRole =
                    userManagementValidator.getPrimaryBusinessRole(primaryRole);

            // Fetch allowed Secondary Role Codes
            String allowedSecondaryRoleCodes =
                    primaryBusinessRole.getSecondaryBrCode();

            log.info("Primary BR Code: {}", primaryBusinessRole.getBrCode());
            log.info("Allowed Secondary BR Codes: {}", allowedSecondaryRoleCodes);
            log.info("Requested Secondary Role: {}", request.getSecondaryRole());

            // Validate allowed Secondary Role
            userManagementValidator.validateAllowedSecondaryRole(
                    request.getSecondaryRole(),
                    allowedSecondaryRoleCodes
            );

            // Fetch selected Secondary Role
            BusinessRolesMaster secondaryRole =
                    userManagementValidator.getSecondaryRole(
                            request.getSecondaryRole()
                    );

            // Validate duplicate assignment
            userManagementValidator.validateDuplicateSecondaryRole(
                    primaryUserId,
                    secondaryUserId,
                    secondaryRole.getId()
            );

            // Save using Mapper
            saveSecondaryUserRole(
                    primaryUserId,
                    secondaryUserId,
                    primaryRole,
                    secondaryRole,
                    request
            );
        }

        log.info("Secondary Role Assignment completed successfully");

    } catch (AdminPortalException e) {
        log.error("Validation failed while assigning Secondary Role", e);
        throw e;

    } catch (Exception e) {
        log.error("Error while assigning Secondary Role", e);
        throw new AdminPortalException(
                GENERIC_ERROR_CODE,
                GENERIC_ERROR_MESSAGE
        );
    }
}


/* =========================================================
   2. UserManagementValidator.java
   ========================================================= */

// Fetch Primary Business Role
public BusinessRolesMaster getPrimaryBusinessRole(UserRole primaryRole) {

    return businessRolesMasterRepository.findById(primaryRole.getRoleId())
            .orElseThrow(() -> new AdminPortalException(
                    GENERIC_ERROR_CODE,
                    PRIMARY_BUSINESS_ROLE_NOT_FOUND
            ));
}


// Validate Allowed Secondary Role
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
                .filter(Objects::nonNull)
                .anyMatch(role ->
                        role.trim().equalsIgnoreCase(
                                requestedSecondaryRole.trim()
                        )
                );

        if (!isAllowed) {
            throw new AdminPortalException(
                    FAILED_ERROR_CODE,
                    SECONDARY_ROLE_NOT_ALLOWED
            );
        }

    } catch (JsonProcessingException e) {
        log.error("Invalid SECONDARY_BR_CODE JSON: {}",
                allowedSecondaryRoleCodes, e);

        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SECONDARY_ROLE_NOT_ALLOWED
        );
    }
}


/* =========================================================
   3. UserManagementDao.java - Save Method
   ========================================================= */

private void saveSecondaryUserRole(
        UUID primaryUserId,
        UUID secondaryUserId,
        UserRole primaryRole,
        BusinessRolesMaster secondaryRole,
        SecondaryRoleAssignmentRequest request) {

    UserRole userRole = userManagementMapper.toSecondaryUserRole(
            primaryUserId,
            secondaryUserId,
            primaryRole,
            secondaryRole,
            request
    );

    userRoleRepository.save(userRole);

    log.info("Secondary Role saved successfully for Primary User: {}",
            primaryUserId);
}


/* =========================================================
   4. UserManagementMapper.java
   ========================================================= */

@Mapping(target = "id", ignore = true)
@Mapping(target = "createdAt", ignore = true)
@Mapping(target = "createdBy", ignore = true)
@Mapping(target = "deletedAt", ignore = true)
@Mapping(target = "userId", source = "primaryUserId")
@Mapping(target = "secondaryUserId", source = "secondaryUserId")
@Mapping(target = "roleId", source = "secondaryRole.id")
@Mapping(target = "aggregatorId", source = "primaryRole.aggregatorId")
@Mapping(target = "isPrimary", constant = "false")
@Mapping(target = "isActive", constant = "true")
@Mapping(target = "fromDate", source = "request.fromDate")
@Mapping(target = "toDate", source = "request.toDate")
@Mapping(target = "remark", source = "request.remarks")
@Mapping(target = "authStatus",
        expression = "java(AuthStatus.PENDING_FOR_AUTHORIZATION)")
@Mapping(target = "recordStatus", constant = "ACTIVE")
UserRole toSecondaryUserRole(
        UUID primaryUserId,
        UUID secondaryUserId,
        UserRole primaryRole,
        BusinessRolesMaster secondaryRole,
        SecondaryRoleAssignmentRequest request
);
