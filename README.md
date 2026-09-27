  #LDAP Config
  ldap:
    # port: 636
    # urls: ldaps://UATROOTDC1.UATAD.SBI:${spring.ldap.port}
    # base: DC=UATAD,DC=SBI
    # username: CN=EPAYUAT_LDAP,OU=Users,OU=Domain Users,DC=UATAD,DC=SBI
    # password: Gitlab@180326
    # port: 636
    # urls: ldaps://corp.ad.sbi:${spring.ldap.port}
    # base: dc=corp, dc=ad, dc=sbi
    # username: cn=cmpicpdevadmin,OU=GTCMP Users,OU=GTCMP,ou=GITC Belapur,dc=corp,dc=ad,dc=sbi
    # password: Cmp0cp@100325
        port: 636
        urls: ldaps://corp.ad.sbi:${spring.ldap.port}
        base: DC=CORP,DC=AD,DC=SBI
        username: CN=EPAY_LDAP,OU=GTAGR Users,OU=GTAGR,OU=GITC Belapur,DC=CORP,DC=AD,DC=SBI
        password: $ldap_test

auth:
  ldap:
    embedded-enabled: false
    user-search-base: ""
    user-search-filter: (sAMAccountName={0})




@RestController
@RequestMapping("/ldap")
@RequiredArgsConstructor
public class LdapTestController {
    private final LdapAuthenticationService ldapAuthenticationService;

    @GetMapping("/test-service-account")
    public String testServiceAccount(@RequestBody LoginRequest loginRequest){
        ldapAuthenticationService.testServiceAccountBind(loginRequest);
        return "LDAP Service account Authentication Success";
    }










public void testServiceAccountBind(LoginRequest loginRequest) {
        try {
            log.info("Testing LDAP service account authentication....");
            contextSource.afterPropertiesSet();
            log.info("LDAP Service user validated, UserDn : {}", contextSource.getUserDn());
            log.info("LDAP Service user validated, PasswordDn : {}", contextSource.getPassword());
            if (contextSource.getUserDn().equals(loginRequest.getUserId())) {
                if (contextSource.getPassword().equals(loginRequest.getPassword())) {
                    contextSource.getContext(contextSource.getUserDn(), contextSource.getPassword());
                    log.info("LDAP Service account Authentication Success");
                }
            } else {
                // throw new AdminPortalException(INCORRECT_AD_ID_ERROR_CODE, INCORRECT_AD_ID_ERROR_MESSAGE);
            }
        } catch (Exception ex) {
            log.error("LDAP Service account authentication failed: {}", ex.getMessage(), ex);
           // throw new AdminPortalException(SERVICE_ACCOUNT_AUTH_FAILED, SERVICE_ACCOUNT_AUTH_ERROR_MESSAGE);
        }
        // ---------------------------------------------------
        DirContext context = null;
        try {
            log.info("Testing LDAP authentication...");
            context = contextSource.getContext(loginRequest.getUserId(), loginRequest.getPassword());
            log.info("LDAP Authentication Success");
        } catch (Exception ex) {
            log.error("LDAP Authentication Failed", ex);
            throw new AdminPortalException(SERVICE_ACCOUNT_AUTH_FAILED, SERVICE_ACCOUNT_AUTH_ERROR_MESSAGE
            );

        } finally {
            if (context != null) {
                try {
                    context.close();
                } catch (Exception ex) {
                    log.info("Failed to close LDAP context", ex);
                }
            }
        }
    }





2026-09-27 10:32:25.485 INFO | com.epay.admin.portal.service.LdapAuthenticationService:86 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=9072e850-6756-4383-9965-685dcb724184 | testServiceAccountBind | Testing LDAP service account authentication....
2026-09-27 10:32:25.486 INFO | com.epay.admin.portal.service.LdapAuthenticationService:88 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=9072e850-6756-4383-9965-685dcb724184 | testServiceAccountBind | LDAP Service user validated, UserDn : CN=EPAY_LDAP,OU=GTAGR Users,OU=GTAGR,OU=GITC Belapur,DC=CORP,DC=AD,DC=SBI
2026-09-27 10:32:25.486 INFO | com.epay.admin.portal.service.LdapAuthenticationService:89 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=9072e850-6756-4383-9965-685dcb724184 | testServiceAccountBind | LDAP Service user validated, PasswordDn : $ldap_test
2026-09-27 10:32:25.486 INFO | com.epay.admin.portal.service.LdapAuthenticationService:105 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=9072e850-6756-4383-9965-685dcb724184 | testServiceAccountBind | Testing LDAP authentication...
2026-09-27 10:32:25.879 ERROR | com.epay.admin.portal.service.LdapAuthenticationService:109 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=9072e850-6756-4383-9965-685dcb724184 | testServiceAccountBind | LDAP Authentication Failed
2026-09-27 10:32:29.150 INFO | com.epay.admin.portal.service.LdapAuthenticationService:86 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=9c9779e7-3910-4305-9f93-fac434752b28 | testServiceAccountBind | Testing LDAP service account authentication....
2026-09-27 10:32:29.150 INFO | com.epay.admin.portal.service.LdapAuthenticationService:88 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=9c9779e7-3910-4305-9f93-fac434752b28 | testServiceAccountBind | LDAP Service user validated, UserDn : CN=EPAY_LDAP,OU=GTAGR Users,OU=GTAGR,OU=GITC Belapur,DC=CORP,DC=AD,DC=SBI
2026-09-27 10:32:29.150 INFO | com.epay.admin.portal.service.LdapAuthenticationService:89 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=9c9779e7-3910-4305-9f93-fac434752b28 | testServiceAccountBind | LDAP Service user validated, PasswordDn : $ldap_test
2026-09-27 10:32:29.151 INFO | com.epay.admin.portal.service.LdapAuthenticationService:105 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=9c9779e7-3910-4305-9f93-fac434752b28 | testServiceAccountBind | Testing LDAP authentication...
2026-09-27 10:32:29.214 ERROR | com.epay.admin.portal.service.LdapAuthenticationService:109 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=9c9779e7-3910-4305-9f93-fac434752b28 | testServiceAccountBind | LDAP Authentication Failed
2026-09-27 10:32:31.678 INFO | com.epay.admin.portal.service.LdapAuthenticationService:86 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=47b9ed0c-64cd-4468-85c8-82bbdacebdbf | testServiceAccountBind | Testing LDAP service account authentication....
2026-09-27 10:32:31.679 INFO | com.epay.admin.portal.service.LdapAuthenticationService:88 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=47b9ed0c-64cd-4468-85c8-82bbdacebdbf | testServiceAccountBind | LDAP Service user validated, UserDn : CN=EPAY_LDAP,OU=GTAGR Users,OU=GTAGR,OU=GITC Belapur,DC=CORP,DC=AD,DC=SBI
2026-09-27 10:32:31.679 INFO | com.epay.admin.portal.service.LdapAuthenticationService:89 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=47b9ed0c-64cd-4468-85c8-82bbdacebdbf | testServiceAccountBind | LDAP Service user validated, PasswordDn : $ldap_test
2026-09-27 10:32:31.680 INFO | com.epay.admin.portal.service.LdapAuthenticationService:105 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=47b9ed0c-64cd-4468-85c8-82bbdacebdbf | testServiceAccountBind | Testing LDAP authentication...
2026-09-27 10:32:31.732 ERROR | com.epay.admin.portal.service.LdapAuthenticationService:109 | principal= | scenario=/api/adminPortal/v1/ldap/test-service-account | operation=GET | correlation=47b9ed0c-64cd-4468-85c8-82bbdacebdbf | testServiceAccountBind | LDAP Authentication Failed
2026-09-27 10:33:05.438 WARN | io.micrometer.common.util.internal.logging.AbstractInternalLogger:170 | principal= | scenario= | operation= | correlation= | log | This Gauge has been already registered (MeterId{name='kafka.consumer.incoming.byte.rate', tags=[tag(client.id=consumer-admin-consumers-1),tag(kafka.version=3.7.2),tag(spring.id=kafkaConsumerFactory.consumer-admin-consumers-1)]}), the Gauge registration will be ignored. Note that subsequent logs will be logged at debug level.
2026-09-27 10:36:44.952 INFO | org.apache.kafka.clients.NetworkClient:997 | principal= | scenario= | operation= | correlation= | handleDisconnections | [AdminClient clientId=adminclient-1] Node -1 disconnected.
