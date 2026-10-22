
SELECT
    RAWTOHEX(ID) AS ID,
    CLIENT_IP,
    DBMS_LOB.GETLENGTH(DEVICE_DETAILS) AS DETAILS_LENGTH,
    UTL_RAW.CAST_TO_VARCHAR2(
        DBMS_LOB.SUBSTR(DEVICE_DETAILS, 2000, 1)
    ) AS DEVICE_DETAILS_JSON
FROM DEVICE_DETAILS
ORDER BY CREATED_DATE DESC;


..........
public AdminPortalResponse<Object> validateOtp(
        OtpValidateRequest otpValidationRequest) {

    // Save Device Details
    DeviceDetailsRequest deviceDetailsRequest =
            otpValidationRequest.getDeviceDetailsRequest();

    if (ObjectUtils.isNotEmpty(deviceDetailsRequest)) {

        DeviceDetailsDto deviceDetailsDto =
                tokenService.buildAdminDeviceInfo(deviceDetailsRequest);

        logger.debug("Device details DTO created. Client IP present: {}",
                ObjectUtils.isNotEmpty(deviceDetailsDto.getClientIp()));

        adminDeviceInfoDao.saveAdminRequestedDeviceInfo(deviceDetailsDto);
    }

    RequestType requestType = RequestType.valueOf(
            otpValidationRequest.getOtpValidationRequest()
                    .getRequestType().toUpperCase());

    return switch (requestType) {
        case LOGIN -> validateAdminOtp(
                otpValidationRequest.getOtpValidationRequest());
        case SIGNUP -> validateMifOtp(
                otpValidationRequest.getOtpValidationRequest());
        default -> throw new IllegalArgumentException(
                "Invalid requestType: " + requestType);
    };
}




,,,,


public DeviceDetailsDto buildAdminDeviceInfo(
        DeviceDetailsRequest deviceDetailsRequest) {

    String deviceDetailsJson =
            AdminPortalUtil.toJson(deviceDetailsRequest);

    return DeviceDetailsDto.builder()
            .clientIp(deviceDetailsRequest.getClientIp())
            .deviceDetails(deviceDetailsJson)
            .build();
}

