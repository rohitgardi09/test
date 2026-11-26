
CREATE TABLE USER_ROLE_AUDIT (
    ID                  RAW(16),
    REV                 NUMBER(19,0) NOT NULL,
    REVTYPE             NUMBER(3,0) NOT NULL,
    AGGREGATOR_ID       RAW(16) NOT NULL,
    ROLE_ID             RAW(16) NOT NULL,
    USER_ID             RAW(16) NOT NULL,
    IS_PRIMARY          NUMBER(1,0) NOT NULL,
    IS_ACTIVE           NUMBER(1,0) NOT NULL,
    CREATED_AT          NUMBER,
    CREATED_BY          VARCHAR2(100 BYTE),
    FROM_DATE           NUMBER,
    TO_DATE             NUMBER,
    SECONDARY_USER_ID   RAW(16),
    REMARK              VARCHAR2(500 BYTE),
    AUTH_STATUS         VARCHAR2(50 BYTE),
    RECORD_STATUS       VARCHAR2(50 BYTE),
    DELETED_AT          NUMBER
);
