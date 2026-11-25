@NoArgsConstructor
@AllArgsConstructor
@Builder
@Data
@Entity
@Table(name = "USER_ROLE")
public class UserRole extends AuditEntityCreatedDate {

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    @Column(nullable = false, updatable = false, unique = true)
    private UUID id;

    private UUID userId;

    private UUID roleId;

    private UUID aggregatorId;

    private Boolean isPrimary;

    private Boolean isActive;

    private Long fromDate;

    private Long toDate;

    @Column(name = "SECONDARY_USER_ID")
    private UUID secondaryUserId;

    private String remark;

    @Enumerated(EnumType.STRING)
    private AuthStatus authStatus;

    private String recordStatus;

    private Long deletedAt;
}




.....

@Repository
public interface UserRoleRepository extends JpaRepository<UserRole, UUID> {

    Optional<UserRole> findByUserIdAndIsPrimaryTrue(UUID userId);

    boolean existsByUserIdAndRoleIdAndSecondaryUserIdAndIsActiveTrue(
            UUID userId,
            UUID roleId,
            UUID secondaryUserId
    );

}




....


@Repository
public interface BusinessRolesMasterRepository
        extends JpaRepository<BusinessRolesMaster, UUID> {

    Optional<BusinessRolesMaster> findByBrCodeAndIsActiveTrue(String brCode);

}




.....


@Query("SELECT u.id FROM Users u WHERE u.adId = :adId")
UUID findIDByAdId(@Param("adId") String adId);



....




public void validateSecondaryRoleAssignments(
        List<SecondaryRoleAssignmentRequest> assignments) {

    if (assignments == null || assignments.isEmpty()) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SECONDARY_ROLE_ASSIGNMENTS_REQUIRED
        );
    }

    for (SecondaryRoleAssignmentRequest request : assignments) {
        validateSecondaryRoleRequest(request);
    }
}

private void validateSecondaryRoleRequest(
        SecondaryRoleAssignmentRequest request) {

    if (request == null) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SECONDARY_ROLE_REQUEST_REQUIRED
        );
    }

    if (request.getPrimaryAdId() == null
            || request.getPrimaryAdId().isBlank()) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                PRIMARY_AD_ID_REQUIRED
        );
    }

    if (request.getSecondaryAdId() == null
            || request.getSecondaryAdId().isBlank()) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SECONDARY_AD_ID_REQUIRED
        );
    }

    if (request.getSecondaryRole() == null
            || request.getSecondaryRole().isBlank()) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SECONDARY_ROLE_REQUIRED
        );
    }

    if (request.getPrimaryAdId().equalsIgnoreCase(
            request.getSecondaryAdId())) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SAME_USER_NOT_ALLOWED
        );
    }

    validateAssignmentDates(
            request.getFromDate(),
            request.getToDate()
    );
}

private void validateAssignmentDates(Long fromDate, Long toDate) {

    if (fromDate == null || toDate == null) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                ASSIGNMENT_DATES_REQUIRED
        );
    }

    if (fromDate > toDate) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                INVALID_ASSIGNMENT_DATES
        );
    }
}



.....



private final UserManagementRepository userManagementRepository;

private final UserRoleRepository userRoleRepository;

private final BusinessRolesMasterRepository businessRolesMasterRepository;

private final UserManagementValidator userManagementValidator;


....


private void updateSecondaryRole(UpdateUserRequest updateUserRequest) {

    try {
        log.info("Starting Secondary Role Assignment");

        List<SecondaryRoleAssignmentRequest> assignments =
                updateUserRequest.getSecondaryRoleAssignment();

        userManagementValidator.validateSecondaryRoleAssignments(assignments);

        for (SecondaryRoleAssignmentRequest request : assignments) {

            UUID primaryUserId = getUserId(request.getPrimaryAdId());

            UUID secondaryUserId = getUserId(request.getSecondaryAdId());

            UserRole primaryRole = getPrimaryRole(primaryUserId);

            BusinessRolesMaster secondaryRole =
                    getSecondaryRole(request.getSecondaryRole());

            validateDuplicateSecondaryRole(
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

    } catch (AdminPortalException e) {
        throw e;

    } catch (Exception e) {
        log.error("Error while assigning Secondary Role", e);

        throw new AdminPortalException(
                GENERIC_ERROR_CODE,
                GENERIC_ERROR_MESSAGE
        );
    }
}



....



private UUID getUserId(String adId) {

    UUID userId = userManagementRepository.findIDByAdId(adId);

    if (userId == null) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                adId.equalsIgnoreCase(adId) ? PRIMARY_USER_NOT_FOUND
                        : SECONDARY_USER_NOT_FOUND
        );
    }

    return userId;
}




.....


private UUID getUserId(String adId, String errorMessage) {

    UUID userId = userManagementRepository.findIDByAdId(adId);

    if (userId == null) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                errorMessage
        );
    }

    return userId;
}





....

UUID primaryUserId = getUserId(
        request.getPrimaryAdId(),
        PRIMARY_USER_NOT_FOUND
);

UUID secondaryUserId = getUserId(
        request.getSecondaryAdId(),
        SECONDARY_USER_NOT_FOUND
);



...

private UserRole getPrimaryRole(UUID primaryUserId) {

    return userRoleRepository
            .findByUserIdAndIsPrimaryTrue(primaryUserId)
            .orElseThrow(() -> new AdminPortalException(
                    FAILED_ERROR_CODE,
                    PRIMARY_ROLE_NOT_ASSIGNED
            ));
}


...


private BusinessRolesMaster getSecondaryRole(String secondaryRoleCode) {

    return businessRolesMasterRepository
            .findByBrCodeAndIsActiveTrue(secondaryRoleCode)
            .orElseThrow(() -> new AdminPortalException(
                    FAILED_ERROR_CODE,
                    SECONDARY_ROLE_NOT_FOUND
            ));
}



....


private void validateDuplicateSecondaryRole(
        UUID primaryUserId,
        UUID secondaryUserId,
        UUID secondaryRoleId) {

    boolean alreadyAssigned =
            userRoleRepository
                    .existsByUserIdAndRoleIdAndSecondaryUserIdAndIsActiveTrue(
                            primaryUserId,
                            secondaryRoleId,
                            secondaryUserId
                    );

    if (alreadyAssigned) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                SECONDARY_ROLE_ALREADY_ASSIGNED
        );
    }
}

private void saveSecondaryUserRole(
        UUID primaryUserId,
        UUID secondaryUserId,
        UserRole primaryRole,
        BusinessRolesMaster secondaryRole,
        SecondaryRoleAssignmentRequest request) {

    UserRole userRole = new UserRole();

    userRole.setUserId(primaryUserId);
    userRole.setSecondaryUserId(secondaryUserId);
    userRole.setRoleId(secondaryRole.getId());

    userRole.setAggregatorId(primaryRole.getAggregatorId());

    userRole.setIsPrimary(false);
    userRole.setIsActive(true);

    userRole.setFromDate(request.getFromDate());
    userRole.setToDate(request.getToDate());
    userRole.setRemark(request.getRemarks());

    userRole.setAuthStatus(AuthStatus.PENDING_FOR_AUTHORIZATION);

    userRole.setRecordStatus("ACTIVE");
    userRole.setDeletedAt(null);

    userRoleRepository.save(userRole);

    log.info(
            "Secondary Role assigned successfully. Primary User ID: {}",
            primaryUserId
    );
}


...




