Authentication authentication =
        SecurityContextHolder.getContext().getAuthentication();

String loggedUser = authentication.getName();

logger.info("Validating logged-in user: {}", loggedUser);

// Check logged-in user's Primary Role
userManagementValidator.validateLoggedInUserPrimaryRole(loggedUser);

// Validate request
userManagementValidator.validateUserCreateRequest(userCreateRequest, roleType);

// Save user
Users users = userManagementDao.saveUser(userCreateRequest);
