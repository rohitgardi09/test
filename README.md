
/* ============================================================
   1. BusinessRolesMasterRepository.java
   ============================================================ */

@Query("SELECT b.secondaryBrCode FROM BusinessRolesMaster b " +
       "WHERE b.brCode = :brCode AND b.isActive = true")
Optional<String> findSecondaryBrCodeByBrCode(
        @Param("brCode") String brCode
);


/* ============================================================
   2. UserManagementValidator.java
   ============================================================ */

// Fetch allowed Secondary Role Codes from BUSINESS_ROLES_MASTER
public String getAllowedSecondaryRoleCodes(String primaryBrCode) {

    return businessRolesMasterRepository
            .findSecondaryBrCodeByBrCode(primaryBrCode)
            .orElseThrow(() -> new AdminPortalException(
                    GENERIC_ERROR_CODE,
                    "Secondary BR Code not found for Primary Role: " + primaryBrCode
            ));
}


// Validate selected Secondary Role
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

    boolean isAllowed = Arrays.stream(allowedSecondaryRoleCodes.split(","))
            .map(String::trim)
            .anyMatch(code ->
                    code.equalsIgnoreCase(requestedSecondaryRole.trim())
            );

    if (!isAllowed) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SECONDARY_ROLE_NOT_ALLOWED
        );
    }
}


/* ============================================================
   3. UserManagementDao.java
      Complete updateSecondaryRole() method
   ============================================================ */

private void updateSecondaryRole(UpdateUserRequest updateUserRequest) {
    try {
        log.info("Starting Secondary Role Assignment");

        List<SecondaryRoleAssignmentRequest> assignments =
                updateUserRequest.getSecondaryRoleAssignment();

        userManagementValidator.validateSecondaryRoleAssignments(assignments);

        for (SecondaryRoleAssignmentRequest request : assignments) {

            // Fetch Primary User ID
            UUID primaryUserId =
                    userManagementValidator.getUserId(request.getPrimaryAdId());

            // Fetch Secondary User ID
            UUID secondaryUserId =
                    userManagementValidator.getUserId(request.getSecondaryAdId());

            // Fetch Primary Role
            UserRole primaryRole =
                    userManagementValidator.getPrimaryRole(primaryUserId);

            // Fetch Primary Business Role
            BusinessRolesMaster primaryBusinessRole =
                    userManagementValidator.getPrimaryBusinessRole(primaryRole);

            // Fetch Primary BR Code
            String primaryBrCode = primaryBusinessRole.getBrCode();

            // Fetch Allowed Secondary Role Codes from Database
            String allowedSecondaryRoleCodes =
                    userManagementValidator.getAllowedSecondaryRoleCodes(primaryBrCode);

            log.info("Primary BR Code: {}", primaryBrCode);
            log.info("Allowed Secondary BR Codes: {}", allowedSecondaryRoleCodes);
            log.info("Requested Secondary Role: {}", request.getSecondaryRole());

            // Validate Allowed Secondary Role
            userManagementValidator.validateAllowedSecondaryRole(
                    request.getSecondaryRole(),
                    allowedSecondaryRoleCodes
            );

            // Fetch Selected Secondary Role
            BusinessRolesMaster secondaryRole =
                    userManagementValidator.getSecondaryRole(
                            request.getSecondaryRole()
                    );

            // Validate Duplicate Secondary Role
            userManagementValidator.validateDuplicateSecondaryRole(
                    primaryUserId,
                    secondaryUserId,
                    secondaryRole.getId()
            );

            // Save Secondary User Role using Mapper
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


/* ============================================================
   4. Required imports
   ============================================================ */

import java.util.Arrays;
import java.util.List;
import java.util.UUID;
import java.util.Optional;

import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
