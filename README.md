
public Users UpdateUserAndAdmins(UpdateUserRequest updateUserRequest) {
    try {
        log.info("Starting User and Admin update for AD ID: {}",
                updateUserRequest.getAdId());

        // Fetch existing user
        Users user = userManagementValidator.getUserByAdId(
                updateUserRequest.getAdId());

        // Update user details
        userManagementMapper.updateUserAndAdmins(updateUserRequest, user);

        // Get primary role code from request
        String primaryRoleCode = updateUserRequest
                .getUserCreationRequest()
                .getPrimaryRoleCode();

        if (primaryRoleCode != null && !primaryRoleCode.isBlank()) {

            // Fetch primary role from BUSINESS_ROLES_MASTER
            BusinessRolesMaster primaryBusinessRole =
                    businessRolesMasterRepository
                            .findByBrCodeAndIsActiveTrue(primaryRoleCode)
                            .orElseThrow(() -> new AdminPortalException(
                                    FAILED_ERROR_CODE,
                                    "Primary Role not found"));

            // Fetch existing primary role entry
            UserRole primaryUserRole = userRoleRepository
                    .findByUserIdAndIsPrimaryTrue(user.getId())
                    .orElseThrow(() -> new AdminPortalException(
                            FAILED_ERROR_CODE,
                            "Primary User Role not found"));

            // Update primary role details
            primaryUserRole.setRoleId(primaryBusinessRole.getId());
            primaryUserRole.setAggregatorId(
                    primaryBusinessRole.getAggregatorId());

            userRoleRepository.save(primaryUserRole);
        }

        Users updatedUser = userManagementRepository.save(user);

        log.info("User and Admin updated successfully for AD ID: {}",
                updateUserRequest.getAdId());

        return updatedUser;

    } catch (AdminPortalException e) {
        throw e;
    } catch (Exception e) {
        log.error("Error while updating User and Admin", e);
        throw new AdminPortalException(
                GENERIC_ERROR_CODE, GENERIC_ERROR_MESSAGE);
    }
}


Optional<UserRole> findByUserIdAndIsPrimaryTrue(UUID userId);
