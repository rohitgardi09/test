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

 

 
