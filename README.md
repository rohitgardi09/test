
Today's payout process was impacted because the "REPORT_FILTER" column in the "REPORT_REQUEST" table of the Operations Service has a maximum size limit of 2500 characters. Today's report filter exceeded 2700+ characters, causing the report processing to fail. As a result, today's payout could not be completed.

We are currently increasing the "REPORT_FILTER" column size in the "REPORT_REQUEST" table from 2500 to 4000 characters. Once this change is deployed, the report will be processed successfully, and the payout execution will resume.





......
...

Today's payout process was impacted because the Operations Service report filter has a current limit of 2500 records, while today's report size exceeded 2700+ records. As a result, the report processing failed, and today's payout could not be completed.

We are currently updating the report filter limit from 2500 to 4000 records. Once this change is deployed, the report will be processed successfully, and the payout execution will resume.









INSERT INTO MERCHANT_USER (
    ID,
    PARENT_USERID,
    USER_NAME,
    FIRST_NAME,
    EMAIL,
    MOBILE_PHONE,
    ROLE_ID,
    STATUS,
    PASSWORD,
    PASSWORD_EXPIRY_TIME,
    LOGIN_FAIL_ATTEMPT,
    LAST_SUCCESS_LOGIN,
    CREATED_BY,
    CREATED_AT,
    UPDATED_BY,
    UPDATED_AT,
    REMARK
) VALUES (
    '163CA03A3A9E4F2C99AA04858F91E049',
    'C3EDEFEACB984971B3081073DE40C139',
    'ROhitG',
    'RohitG',
    'rohit@test.com',
    '7387439091',
    '45462CCE1D8E12BAE0630487B10A4D4A',
    'ACTIVE',
    '7tlpKNgg0t6SDyKUmIQUV3wAafh4ARog+Ake1ELTarc8k6JnVWfKAVoQM3riBCAv6rKtP8I1OhaC+RkKMxceig==',
    1790143660703,
    0,
    1782205657500,
    'RohitG',
    1770966904305,
    'anonymousUser',
    1782205657502,
    'verified user'
);
