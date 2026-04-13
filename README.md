CREATE TABLE MASTER_PURGE_LOG_DETAILS (
    job_id        NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    proc_name     VARCHAR2(200),
    start_time    TIMESTAMP,
    end_time      TIMESTAMP,
    created_by VARCHAR2(200),
    updated_by VARCHAR2(200),
    status        VARCHAR2(20),
    error_message VARCHAR2(4000)
);


create or replace PROCEDURE SP_PURGE_EXPIRED_TOKEN
AS
    BATCH_SIZE CONSTANT NUMBER := 5000;
    TYPE t_id_list IS TABLE OF TOKEN.ID%TYPE;
    v_id_list t_id_list;
    v_cutoff_time number;
    v_job_id        NUMBER;
    dml_errors EXCEPTION;
    PRAGMA EXCEPTION_INIT(dml_errors, -24381);
    BEGIN
        INSERT INTO MASTER_PURGE_LOG_DETAILS (proc_name, start_time, status,created_by,updated_by)
        VALUES ('SP_PURGE_EXPIRED_TOKEN', SYSTIMESTAMP, 'STARTED',USER,USER)
        RETURNING job_id INTO v_job_id;
        COMMIT;
    SELECT (CAST(SYSTIMESTAMP AT TIME ZONE 'UTC' AS DATE) - DATE '1970-01-01') * 86400000
    INTO v_cutoff_time
    FROM dual;
    LOOP
        SELECT id
        BULK COLLECT INTO v_id_list
        FROM token
        WHERE token_expiry_time < v_cutoff_time
        ORDER BY token_expiry_time
        FETCH FIRST BATCH_SIZE ROWS ONLY;
        EXIT WHEN v_id_list.COUNT = 0;
		 BEGIN
       FORALL i IN 1 .. v_id_list.COUNT SAVE EXCEPTIONS
                DELETE FROM token
                WHERE id = v_id_list(i);
                COMMIT;
      EXCEPTION
            WHEN dml_errors THEN
               DBMS_OUTPUT.PUT_LINE ('Error during bulk delete');
                  END;
                  END LOOP;
                  UPDATE MASTER_PURGE_LOG_DETAILS SET STATUS='SUCCESS',
                  END_TIME=SYSTIMESTAMP
                  WHERE JOB_ID=V_JOB_ID;
                  COMMIT;

				  EXCEPTION WHEN OTHERS THEN
				  UPDATE MASTER_PURGE_LOG_DETAILS SET STATUS='FAILED',
                  END_TIME=SYSTIMESTAMP
                  WHERE JOB_ID=V_JOB_ID;
                  COMMIT;
				  RAISE;
                  END;


                  BEGIN
  DBMS_SCHEDULER.CREATE_JOB (
    job_name      => 'JOB_PURGE_EXPIRED_TOKEN',
    job_type      => 'STORED_PROCEDURE',
    job_action    => 'SP_PURGE_EXPIRED_TOKEN',
    start_date    => SYSTIMESTAMP,
    repeat_interval => 'FREQ=DAILY;BYHOUR=2;BYMINUTE=0;BYSECOND=0',
    enabled       => TRUE,
    comments      => 'Job to execute the daily procedure at 2 AM to delete expired tokens'
  );
END;
/
