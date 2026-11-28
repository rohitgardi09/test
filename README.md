public AdminPortalResponse<userCreationResponse> CreateUser(
        UserCreationRequest userCreateRequest, String roleType) {

    String loggedUser =
            EPayIdentityUtil.getUserPrincipal().getUsername();

    logger.info("Validating logged-in user: {}", loggedUser);

    // Check logged-in user's Primary Role
    userManagementValidator.validateLoggedInUserPrimaryRole(loggedUser);

    logger.info("Validating user info for AD ID: {}",
            userCreateRequest.getAdId());

    userManagementValidator.validateUserCreateRequest(
            userCreateRequest, roleType);

    logger.info("Saving user info for AD ID: {}",
            userCreateRequest.getAdId());

    Users users = userManagementDao.saveUser(userCreateRequest);

    userCreationResponse userResponse =
            userCreationResponse.builder()
                    .userId(users.getId())
                    .message(USER_CREATED_SUCCESSFULLY)
                    .build();

    logger.info("Successfully saved user details for userId: {}",
            users.getId());

    return AdminPortalResponse.builder()
            .status(SUCCESS_RESPONSE_CODE)
            .data(List.of(userResponse))
            .build();
}

public void validateLoggedInUserPrimaryRole(String loggedUser) {

    UUID loggedUserId = userManagementRepository.findIDByAdId(loggedUser);

    if (loggedUserId == null) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                "Logged-in user not found"
        );
    }

    boolean hasPrimaryRole =
            userRoleRepository.existsByUserIdAndIsPrimaryTrueAndIsActiveTrue(
                    loggedUserId
            );

    if (!hasPrimaryRole) {
        throw new AdminPortalException(
                FAILED_ERROR_CODE,
                "You are not authorized to create users. Primary role is required."
        );
    }
}


boolean existsByUserIdAndIsPrimaryTrueAndIsActiveTrue(UUID userId); 

