package com.epay.admin.portal.config;

import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;
import com.unboundid.ldap.listener.InMemoryDirectoryServer;
import com.unboundid.ldap.listener.InMemoryDirectoryServerConfig;
import com.unboundid.ldap.listener.InMemoryListenerConfig;
import com.unboundid.ldif.LDIFReader;

import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Configuration;

import java.io.InputStream;

@Configuration
@ConditionalOnProperty(
        name = "auth.ldap.embedded-enabled",
        havingValue = "true"
)
public class EmbeddedLdapServerConfig {

    private final LoggerUtility logger =
            LoggerFactoryUtility.getLogger(this.getClass());

    public static final String FILENAME = "users.ldif";

    @Value("${spring.ldap.base}")
    private String baseDn;

    @Value("${spring.ldap.port:8389}")
    private int port;

    private InMemoryDirectoryServer directoryServer;

    @PostConstruct
    public void startLdapServer() throws Exception {

        logger.info("Initializing Embedded LDAP server...");

        InMemoryDirectoryServerConfig config =
                new InMemoryDirectoryServerConfig(baseDn);

        config.setSchema(null);

        config.setListenerConfigs(
                InMemoryListenerConfig.createLDAPConfig(
                        "default",
                        port
                )
        );

        try (InputStream inputStream =
                     getClass().getClassLoader()
                             .getResourceAsStream(FILENAME)) {

            if (inputStream == null) {
                throw new IllegalStateException(
                        FILENAME + " not found in src/main/resources"
                );
            }

            directoryServer =
                    new InMemoryDirectoryServer(config);

            directoryServer.importFromLDIF(
                    true,
                    new LDIFReader(inputStream)
            );

            directoryServer.startListening();

            logger.info(
                    "Embedded LDAP started successfully on port: {}",
                    port
            );

        } catch (Exception e) {

            logger.error(
                    "Failed to start embedded LDAP server",
                    e
            );

            if (directoryServer != null) {
                directoryServer.shutDown(true);
                directoryServer = null;
            }

            throw new IllegalStateException(
                    "Unable to start embedded LDAP server on port: "
                            + port,
                    e
            );
        }
    }

    @PreDestroy
    public void stopLdapServer() {

        if (directoryServer != null) {
            logger.info("Stopping embedded LDAP server...");

            directoryServer.shutDown(true);
            directoryServer = null;
        }
    }
}
