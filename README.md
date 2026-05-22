Hi Team,
I have completed the centralized login, but project access is still not available.
If anyone else is also facing the same issue, please connect with DevOps.





@Transactional
    public void processUsersWithExpiredPassword() {
        logger.info("Find and update Expired pwd - Scheduler start.");
        long currentTimeStamp = Instant.now().toEpochMilli();
        logger.info("Current timestamp: "+ currentTimeStamp);

        //1- Get list of Merchant users who passed pwd expiration time.
        List<MerchantUser> merchantUsers = merchantUserRepository.findAllByPasswordExpiryTimeLessThanAndStatusNotIn(currentTimeStamp, List.of(UserStatus.EXPIRED, UserStatus.BLOCKED, UserStatus.ACTIVE, UserStatus.INACTIVE));

        //2- Check collection for users.
        if(!merchantUsers.isEmpty()) {
            logger.info("Total merchant users: "+merchantUsers.size());
            batchUpdateForPasswordExpiry(merchantUsers);
        } else {
            logger.info("Zero users found with expired pwd - Scheduler End. ");
        }
    }

----------------------------------------------------------------------------------------------------------------------------
	
	MerchantUserRepository
	
	
	List<MerchantUser> findAllByPasswordExpiryTimeLessThanAndStatusNotIn(Long currentTime, List<UserStatus> userStatus);
	
----------------------------------------------------------------------------------------------------------------------------
	PasswordManagementDao
	
	
	/**
     * Set status as auto expired
     *
     * @param merchantUsers - List of merchant users pwd passed expiry date.
     */
    private void batchUpdateForPasswordExpiry(List<MerchantUser> merchantUsers) {
        List<PasswordManagement> passwordHistory = new ArrayList<>();

        merchantUsers.forEach( merchantUser -> {
            long currTimeStamp = Instant.now().toEpochMilli();
            PasswordManagement  mph = PasswordManagement.builder()
                    .userId(merchantUser.getId())
                    .status(PasswordStatusType.EXPIRED)
                    .previousPassword(merchantUser.getPassword())
                    .requestType(PasswordManagementType.AUTO_EXPIRE_PASSWORD)
                    .createdAt(merchantUser.getCreatedAt())
                    .updatedAt(currTimeStamp)
                    .build();
            passwordHistory.add(mph);

            merchantUser.setStatus(UserStatus.EXPIRED);
            merchantUser.setUpdatedAt(currTimeStamp);
        });

        if(!passwordHistory.isEmpty()) {
            // Save pwd history.
            logger.info("pwd history records: "+passwordHistory.size());
            passwordManagementRepository.saveAllAndFlush(passwordHistory);
        }
        // Update merchant users (expired pwd)
        merchantUserRepository.saveAllAndFlush(merchantUsers);
    }
