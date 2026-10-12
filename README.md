@Test
void sendEmailNotification_success() {
    AdminPortalEmailDto deviationEmailDto =
            AdminPortalEmailDto.builder().build();

    EmailDto emailDto = EmailDto.builder().build();

    when(notificationMapper.mapEmailDtoToDto(deviationEmailDto))
            .thenReturn(emailDto);

    doNothing().when(emailService).sendEmail(emailDto);

    notificationDao.sendEmailNotification(deviationEmailDto);

    verify(notificationMapper, times(1))
            .mapEmailDtoToDto(deviationEmailDto);

    verify(notificationManagementRepository, times(1))
            .save(any(NotificationManagement.class));
}
