public Users saveUser(UserCreationRequest userCreateRequest) {
    try {
        // Create user entity
        Users user = userManagementMapper.mapRequestToEntity(userCreateRequest);

        // Set aggregator
        Optional<UUID> aggregatorId =
                aggregatorMasterRepository.findIdByAggregatorCode(SBI_AGGREGATOR_CODE);

        aggregatorId.ifPresent(user::setAggregatorId);

        user.setStatus(UserStatus.ACTIVE);
        user.setAuthStatus(AuthStatus.PENDING);

        // Save user first to generate User ID
        Users savedUser = userManagementRepository.save(user);

        // Fetch Primary Role using BR_CODE
        String primaryRoleCode = userCreateRequest.getPrimaryRoleCode();

        if (primaryRoleCode == null || primaryRoleCode.isBlank()) {
            throw new AdminPortalException(
                    FAILED_ERROR_CODE, "Primary Role Code is mandatory");
        }

        BusinessRolesMaster primaryRole = businessRolesMasterRepository
                .findByBrCodeAndIsActiveTrue(primaryRoleCode)
                .orElseThrow(() -> new AdminPortalException(
                        FAILED_ERROR_CODE, "Primary Role not found"));

        // Create Primary User Role entry
        UserRole userRole = UserRole.builder()
                .userId(savedUser.getId())
                .roleId(primaryRole.getId())
                .aggregatorId(primaryRole.getAggregatorId())
                .isPrimary(true)
                .isActive(true)
                .authStatus(AuthStatus.PENDING_FOR_AUTHORIZATION)
                .recordStatus(1)
                .build();

        userRoleRepository.save(userRole);

        return savedUser;

    } catch (AdminPortalException e) {
        throw e;
    } catch (Exception e) {
        log.error("Error while saving user information", e);
        throw new AdminPortalException(
                GENERIC_ERROR_CODE, GENERIC_ERROR_MESSAGE);
    }
}
