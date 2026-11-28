{
  "adId": "LOGGED_IN_ADMIN_AD_ID",
  "userCreationRequest": {
    "adId": "PRIMARY_USER_AD_ID"
  },
  "secondaryRoleAssignment": {
    "secondaryRoleCode": "RELATIONSHIP_MANAGER",
    "secondaryAdId": "SECONDARY_USER_AD_ID",
    "fromDate": 1795824000000,
    "toDate": 1827360000000,
    "remarks": "Secondary role assignment"
  },
  "relievedStatus": false
}






@Mapping(target = "firstName", source = "userCreationRequest.firstName")
@Mapping(target = "lastName", source = "userCreationRequest.lastName")
@Mapping(target = "designation", source = "userCreationRequest.designation")
@Mapping(target = "phoneNumber", source = "userCreationRequest.mobilePhone")
@Mapping(target = "emailId", source = "userCreationRequest.emailId")
@Mapping(target = "officeAddress", source = "userCreationRequest.officeAddress")
@Mapping(target = "authStatus", expression = "java(AuthStatus.PENDING)")
void updateUserAndAdmins(
        UpdateUserRequest updateUserRequest,
        @MappingTarget Users user
);



public Users updateUserAndAdmins(UpdateUserRequest updateUserRequest) {
    try {
        log.info("Starting User and Admin update for AD ID: {}",
                updateUserRequest.getAdId());

        Users user = userManagementValidator
                .getUserByAdId(updateUserRequest.getAdId());

        userManagementMapper.updateUserAndAdmins(updateUserRequest, user);

        Users updatedUser = userManagementRepository.save(user);

        log.info("User and Admin updated successfully for AD ID: {}",
                updateUserRequest.getAdId());

        return updatedUser;

    } catch (AdminPortalException e) {
        log.error("Error while updating User and Admin", e);
        throw e;
    } catch (Exception e) {
        log.error("Error while updating User and Admin", e);
        throw new AdminPortalException(GENERIC_ERROR_CODE, GENERIC_ERROR_MESSAGE);
    }
}



