spring_boot=3.5.8
dependency_plugin=1.1.7

oracle_driver=23.5.0.24.07

mapstruct=1.5.1.Final
lombok_mapstruct=0.2.0

commons_lang3=3.18.0


plugins {
    id 'java'
    id 'org.springframework.boot' version "${spring_boot}"
    id 'io.spring.dependency-management' version "${dependency_plugin}"
}

repositories {
    mavenCentral()
}

dependencies {

    implementation 'org.springframework.boot:spring-boot-starter-web'

    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

    implementation 'org.springframework.boot:spring-boot-starter-aop'

    implementation "com.oracle.database.jdbc:ojdbc11:${oracle_driver}"

    implementation "org.apache.commons:commons-lang3:${commons_lang3}"

    implementation 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    implementation "org.mapstruct:mapstruct:${mapstruct}"
    annotationProcessor "org.mapstruct:mapstruct-processor:${mapstruct}"

    implementation "org.projectlombok:lombok-mapstruct-binding:${lombok_mapstruct}"

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}




end 





file name - gradle.properties
version=0.0.1
spring_boot=3.5.8
dependency_plugin=1.1.7
spring_boot_devtools=3.3.4
spring_context=5.3.30
swagger=2.8.13
javax_persistence=2.2
oracle_driver=23.5.0.24.07
shedlock=5.9.0
commons_io=2.18.0
itext=9.0.0
mapstruct=1.5.1.Final
lombok_mapstruct=0.2.0
jwt=0.11.5
javax_mail=1.6.2
thymeleaf=3.1.3.RELEASE
logback=1.5.13
jhlabs=2.0.235-1
commons_csv=1.12.0
freeTTS=1.2.2
flying_saucer_pdf=9.11.3
sbi_logging=1.0.0
sbi_crypto=1.0.0
sbi_auth=1.0.0
sbi_notification=1.0.0
sbi_captcha=1.0.0
jackson_blackbird=2.20.1
commons_lang3=3.18.0

************************************************************************************

file name - build.gradle

plugins {
    id 'java'
    id 'org.springframework.boot' version "${spring_boot}"
    id 'io.spring.dependency-management' version "${dependency_plugin}"
}

group = 'com.epay.merchant'
version = "${version}"

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

repositories {
    mavenCentral()
    flatDir {
        dirs "libs"
    }
    maven {
        url "https://gitlab.epay.sbi/api/v4/projects/16/packages/maven"
        credentials(PasswordCredentials) {
            username = project.findProperty("gitlab.username")?: System.getenv("CI_USERNAME")
            password = project.findProperty("gitlab.token")?: System.getenv("CI_JOB_TOKEN")
        }
        authentication {
            basic(BasicAuthentication)
        }
    }
}

dependencies {
    implementation "com.fasterxml.jackson.module:jackson-module-blackbird:${jackson_blackbird}"
    implementation "org.apache.commons:commons-lang3:${commons_lang3}"
    implementation 'org.springframework.boot:spring-boot-starter-webflux'
    implementation 'org.springframework.boot:spring-boot-starter-json'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-mail'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-aop'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation "org.springframework.boot:spring-boot-devtools:${spring_boot_devtools}"
    implementation "org.springframework:spring-context-support:${spring_context}"
    implementation "org.springdoc:springdoc-openapi-starter-webmvc-ui:${swagger}"

    implementation "javax.persistence:javax.persistence-api:${javax_persistence}"
    implementation "org.hibernate.orm:hibernate-envers"
    implementation "com.oracle.database.jdbc:ojdbc11:${oracle_driver}"
    implementation 'org.liquibase:liquibase-core'

    implementation "net.javacrumbs.shedlock:shedlock-spring:${shedlock}"
    implementation "net.javacrumbs.shedlock:shedlock-provider-jdbc-template:${shedlock}"

    implementation "commons-io:commons-io:${commons_io}"

    implementation "com.itextpdf:itext-core:${itext}"
    implementation "com.itextpdf:bouncy-castle-adapter:${itext}"

    implementation 'org.springframework.kafka:spring-kafka'
    //keep lombok then mapstruct
    implementation 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    implementation "org.mapstruct:mapstruct:${mapstruct}"
    annotationProcessor "org.mapstruct:mapstruct-processor:${mapstruct}"
    implementation "org.projectlombok:lombok-mapstruct-binding:${lombok_mapstruct}"

    //Utility
    implementation "com.sbi.epay:logging-service:${sbi_logging}"
    implementation "com.sbi.epay:encryption-decryption-service:${sbi_crypto}"
    implementation "com.sbi.epay:authentication-service:${sbi_auth}"
    implementation "com.sbi.epay:notification-service:${sbi_notification}"
    implementation "com.sbi.epay:captcha-service:${sbi_captcha}"

    implementation "net.sf.sociaal:freetts:${freeTTS}"

    implementation "javax.mail:javax.mail-api:${javax_mail}"
    implementation "com.sun.mail:javax.mail:${javax_mail}"
    implementation "org.thymeleaf:thymeleaf:${thymeleaf}"
    implementation "org.thymeleaf:thymeleaf-spring5:${thymeleaf}"
    implementation "org.xhtmlrenderer:flying-saucer-pdf:${flying_saucer_pdf}"

    implementation "com.jhlabs:filters:${jhlabs}"
    implementation "org.apache.commons:commons-csv:${commons_csv}"
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
    testCompileOnly "org.mapstruct:mapstruct:${mapstruct}"

}

configurations {
    all*.exclude module : 'spring-boot-starter-logging'
    all*.exclude module : 'slf4j-simple'
}

tasks.withType(JavaExec).configureEach {
    jvmArgs(['--add-opens=java.base/java.lang=ALL-UNNAMED'])
}

tasks.named('test') {
    useJUnitPlatform()
}
springBoot  {
    buildInfo()
}
bootJar {
    duplicatesStrategy(DuplicatesStrategy.EXCLUDE)
}

























Exception Tracker 

TrackException.java 
@Target(ElementType.METHOD) 
@Retention(RetentionPolicy.RUNTIME) 
@Documented 
public @interface TrackException { 
 
    boolean storeStackTrace() default true; 
 
    boolean rethrow() default true; 
} 

ExceptionLog.java 
@Entity 
@Table(name = "EXCEPTION_LOG") 
@Getter 
@Setter 
@Builder 
@NoArgsConstructor 
@AllArgsConstructor 
public class ExceptionLog { 
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY) 
    private Long id; 
    private String serviceName; 
    private String className; 
    private String methodName; 
    private String exceptionType; 
    @Lob 
    private String exceptionMessage; 
    @Lob 
    private String stackTrace; 
    private String mid; 
    private String orderRef; 
    private String atrn; 
    private String paymode; 
    private String traceId; 
    @Lob 
    private String mdcJson; 
    private Long created; 
} 

ExceptionDTO.java 

@Getter 
@Setter 
@Builder 
@NoArgsConstructor 
@AllArgsConstructor 
public class ExceptionDTO { 
    private String serviceName; 
    private String className; 
    private String methodName; 
    private String exceptionType; 
    private String exceptionMessage; 
    private String stackTrace; 
    private Map<String, String> mdcMap; 
  } 

 

ExceptionLogRepository.java 

public interface ExceptionLogRepository extends JpaRepository<ExceptionLog, Long> { 
} 

 

MDCUtil.java 

public class MDCUtil { 
 
    public static String getIgnoreCase( Map<String, String> map, String... keys) { 
 
        if (map == null) { 
            return "”; 
        } 
 
        for (String key : keys) { 
            for (Map.Entry<String, String> entry : map.entrySet()) { 
                if (entry.getKey().equalsIgnoreCase(key)) { 
                    return entry.getValue(); 
                } 
            } 
        } 
        return "”; 
    } 
} 

StackTraceUtil.java 

public class StackTraceUtil { 
    public static String getShortStackTrace(Throwable ex) { 
        StringBuilder sb = new StringBuilder(); 
        sb.append(ex.getMessage()).append("\n"); 
        StackTraceElement[] elements = ex.getStackTrace(); 
        int limit = Math.min(elements.length, 10); 
        for (int i = 0; i < limit; i++)  

{ 
            sb.append(elements[i].toString()) 
              .append("\n"); 
        } 
        return sb.toString(); 
    } 
} 

 

AsyncConfig.java 
@Configuration 
@EnableAsync 
public class AsyncConfig { 
 
    @Bean("exceptionExecutor") 
    public Executor exceptionExecutor() { 
 
        ThreadPoolTaskExecutor executor = 
                new ThreadPoolTaskExecutor(); 
 
        executor.setCorePoolSize(2); 
        executor.setMaxPoolSize(10); 
        executor.setQueueCapacity(1000); 
        executor.setThreadNamePrefix("exception-"); 
 
        executor.initialize(); 
 
        return executor; 
    } 
} 

 

 

 

ExceptionAsyncService.java 

@Service 
@RequiredArgsConstructor 
@Slf4j 
public class ExceptionAsyncService { 
 
    private final ExceptionLogRepository exceptionLogRepository; 
    private final ExceptionSearchIndexRepository searchRepository; 
    private final ObjectMapper objectMapper; 
 
    @Async("exceptionExecutor") 
    public void store(ExceptionDTO dto) { 
 
        try { 
 
            Map<String, String> mdc = dto.getMdcMap(); 
 
            ExceptionLog entity = 
                    ExceptionLog.builder() 
                            .serviceName(dto.getServiceName()) 
                            .className(dto.getClassName()) 
                            .methodName(dto.getMethodName()) 
                            .exceptionType(dto.getExceptionType()) 
                            .exceptionMessage(dto.getExceptionMessage()) 
                            .stackTrace(dto.getStackTrace()) 
                            .mid(MDCUtil.getIgnoreCase(mdc, "MID")) 
                            .orderRef( 
                                    MDCUtil.getIgnoreCase( 
                                            mdc, 
                                            "ORDER_REF", 
                                            "ORDERID")) 
                            .atrn( 
                                    MDCUtil.getIgnoreCase( 
                                            mdc, 
                                            "ATRN")) 
                            .paymode( 
                                    MDCUtil.getIgnoreCase( 
                                            mdc, 
                                            "PAYMODE")) 
                            .traceId( 
                                    MDCUtil.getIgnoreCase( 
                                            mdc, 
                                            "TRACE_ID")) 
                            .requestId( 
                                    MDCUtil.getIgnoreCase( 
                                            mdc, 
                                            "REQUEST_ID")) 
                            .mdcJson(objectMapper.writeValueAsString(mdc)) 
                            .createdAt(dto.getCreatedAt()) 
                            .build(); 
 
            exceptionLogRepository.save(entity); 
 
 
        } catch (Exception ex) { 
 
            log.error("Failed to store exception", ex); 
        } 
    } 
} 

 

ExceptionTrackerAspect.java 

@Aspect 
@Component 
@RequiredArgsConstructor 
@Slf4j 
public class ExceptionTrackerAspect { 
 
    private final ExceptionAsyncService asyncService; 
 
    @Value("${spring.application.name}") 
    private String serviceName; 
 
    @Around("@annotation(trackException)") 
    public void trackException( 
            ProceedingJoinPoint joinPoint, 
            TrackException trackException) 
            throws Throwable { 
 
        try { 
 
            return joinPoint.proceed(); 
 
        } catch (Exception ex) { 
 
            MethodSignature signature = 
                    (MethodSignature) 
                            joinPoint.getSignature(); 
 
            Map<String, String> mdc = 
                    MDC.getCopyOfContextMap(); 
 
            ExceptionDTO dto = 
                    ExceptionDTO.builder() 
                            .serviceName(serviceName) 
                            .className( 
                                    signature 
                                            .getDeclaringTypeName()) 
                            .methodName( 
                                    signature 
                                            .getMethod() 
                                            .getName()) 
                            .exceptionType( 
                                    ex.getClass().getName()) 
                            .exceptionMessage( 
                                    ex.getMessage()) 
                            .stackTrace( 
                                    trackException.storeStackTrace() 
                                            ? StackTraceUtil 
                                            .getShortStackTrace(ex) 
                                            : null) 
                            .mdcMap(mdc) 
                            .createdAt(LocalDateTime.now()) 
                            .build(); 
 
            asyncService.store(dto); 
 
            
           } 
    } 
} 

 

PaymentService.java 

package com.company.payment.service; 
 
import com.company.exceptiontracker.annotation.TrackException; 
import lombok.extern.slf4j.Slf4j; 
import org.slf4j.MDC; 
import org.springframework.stereotype.Service; 
 
@Service 
@Slf4j 
public class PaymentService { 
 
    @TrackException 
  public void processPayment() { 
try { 

int x = 10 / 0; 

} catch (Exception ex) { 

log.error( "Payment Processing Failed", ex); 

throw ex; 

} 
     } 
} 

 

How It Works 

PaymentService 
        | 
        v 
@TrackException 
        | 
        v 
AOP Aspect Intercepts 
        | 
        +--> Reads MDC 
        +--> Reads Exception 
        +--> Reads Class Name 
        +--> Reads Method Name 
        +--> Async Store 
        | 
        v 
Oracle DB 

 

Required Oracle Tables 

CREATE TABLE EXCEPTION_LOG ( 
 
    ID NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY, 
 
    SERVICE_NAME VARCHAR2(100), 
 
    CLASS_NAME VARCHAR2(500), 
 
    METHOD_NAME VARCHAR2(500), 
 
    EXCEPTION_TYPE VARCHAR2(500), 
 
    EXCEPTION_MESSAGE CLOB, 
 
    STACK_TRACE CLOB, 
 
    MID VARCHAR2(100), 
 
    ORDER_REF VARCHAR2(200), 
 
    ATRN VARCHAR2(200), 
 
    PAYMODE VARCHAR2(100), 
 
    TRACE_ID VARCHAR2(200), 
 
    REQUEST_ID VARCHAR2(200), 
 
    MDC_JSON CLOB, 
 
    CREATED_AT TIMESTAMP 
); 

 

 
