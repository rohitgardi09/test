
-- =========================================
-- 1. ADD COLUMNS IN BUSINESS_ROLES_MASTER
-- =========================================

ALTER TABLE BUSINESS_ROLES_MASTER
ADD (
    USER_TYPE VARCHAR2(50),
    SECONDARY_BR_CODES VARCHAR2(2000)
);


-- =========================================
-- 2. UPDATE USER_TYPE
-- =========================================

UPDATE BUSINESS_ROLES_MASTER
SET USER_TYPE =
    CASE
        WHEN BR_CODE = 'MASTER_ADMIN' THEN 'Master_Admin'
        WHEN BR_CODE = 'SUPER_ADMIN' THEN 'Super_Admin'
        WHEN BR_CODE IN (
            'BIZ_GROUP_ADMIN',
            'BANK_IT_GROUP_ADMIN',
            'OPS_ITDCR_GROUP_ADMIN',
            'CMD_GROUP_ADMIN'
        ) THEN 'Group_Admin'
        WHEN TYPE = 'USER' THEN 'User'
        ELSE 'Admin'
    END;


-- =========================================
-- 3. SECONDARY ROLE MAPPING
-- =========================================

UPDATE BUSINESS_ROLES_MASTER
SET SECONDARY_BR_CODES =
    'RELATIONSHIP_MANAGER,ONBOARDING_MANAGER'
WHERE BR_CODE = 'CHANNEL_MANAGER';

UPDATE BUSINESS_ROLES_MASTER
SET SECONDARY_BR_CODES = 'CHANNEL_MANAGER'
WHERE BR_CODE = 'RELATIONSHIP_MANAGER';

UPDATE BUSINESS_ROLES_MASTER
SET SECONDARY_BR_CODES = 'CHANNEL_MANAGER'
WHERE BR_CODE = 'ONBOARDING_MANAGER';

UPDATE BUSINESS_ROLES_MASTER
SET SECONDARY_BR_CODES = 'ONBOARDING_MANAGER'
WHERE BR_CODE = 'BIZ_SUPPORT';


-- =========================================
-- 4. VERIFY DATA
-- =========================================

SELECT
    ID,
    BR_CODE,
    NAME,
    TYPE,
    USER_TYPE,
    SECONDARY_BR_CODES,
    IS_ACTIVE
FROM BUSINESS_ROLES_MASTER
ORDER BY BR_CODE;

COMMIT;



/* =========================================================
   1. BusinessRolesMaster.java
   Add these fields in existing Entity
   ========================================================= */

@Column(name = "USER_TYPE")
private String userType;

@Column(name = "SECONDARY_BR_CODES")
private String secondaryBrCodes;


/* =========================================================
   2. BusinessRolesMasterRepository.java
   Add this method in existing Repository
   ========================================================= */

List<BusinessRolesMaster> findByBrCodeInAndIsActiveTrue(
        List<String> brCodes
);


/* =========================================================
   3. AdminPortalConstants.java
   Add this constant
   ========================================================= */

public static final String SECONDARY_ROLE_NOT_ALLOWED =
        "Selected Secondary Role is not allowed for this Primary Role";


/* =========================================================
   4. UserManagementValidator.java
   Add this validation method
   ========================================================= */

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

    boolean isAllowed = Arrays.stream(
                    allowedSecondaryRoleCodes.split(","))
            .map(String::trim)
            .anyMatch(code ->
                    code.equalsIgnoreCase(
                            requestedSecondaryRole.trim()
                    )
            );

    if (!isAllowed) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SECONDARY_ROLE_NOT_ALLOWED
        );
    }
}


/* =========================================================
   5. UserManagementDao.java
   Add this method to get allowed Secondary Roles
   ========================================================= */

public List<BusinessRolesMaster> getAllowedSecondaryRoles(
        String primaryAdId) {

    UUID primaryUserId =
            userManagementRepository.findIDByAdId(primaryAdId);

    if (primaryUserId == null) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                PRIMARY_USER_NOT_FOUND
        );
    }

    UserRole primaryRole =
            userRoleRepository
                    .findByUserIdAndIsPrimaryTrue(primaryUserId)
                    .orElseThrow(() -> new AdminPortalException(
                            FAILED_ERROR_CODE,
                            PRIMARY_ROLE_NOT_ASSIGNED
                    ));

    BusinessRolesMaster primaryBusinessRole =
            businessRolesMasterRepository
                    .findById(primaryRole.getRoleId())
                    .orElseThrow(() -> new AdminPortalException(
                            FAILED_ERROR_CODE,
                            PRIMARY_ROLE_NOT_ASSIGNED
                    ));

    String secondaryBrCodes =
            primaryBusinessRole.getSecondaryBrCodes();

    if (secondaryBrCodes == null || secondaryBrCodes.isBlank()) {
        return Collections.emptyList();
    }

    List<String> allowedCodes =
            Arrays.stream(secondaryBrCodes.split(","))
                    .map(String::trim)
                    .filter(code -> !code.isEmpty())
                    .toList();

    return businessRolesMasterRepository
            .findByBrCodeInAndIsActiveTrue(allowedCodes);
}


/* =========================================================
   6. updateSecondaryRole() - Existing DAO method
   Add this validation after fetching Primary Business Role
   ========================================================= */

userManagementValidator.validateAllowedSecondaryRole(
        request.getSecondaryRole(),
        primaryBusinessRole.getSecondaryBrCodes()
);


/* =========================================================
   7. Required imports
   Add only if not already present
   ========================================================= */

import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.UUID;

import com.epay.admin.portal.entity.admin.BusinessRolesMaster;
import com.epay.admin.portal.entity.admin.UserRole;



