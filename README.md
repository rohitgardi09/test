private LdapSearchDto mapToLdapSearchDto(UserMViewDto userMViewDto) {
    return LdapSearchDto.builder()
            .adId(userMViewDto.getAdId())
            .firstName(userMViewDto.getFirstName())
            .lastName(userMViewDto.getLastName())
            .emailId(userMViewDto.getEmailId())
            .build();
}
