void test_SendSms_Notification_success() {
        MerchantSmsDto merchantSmsDto = MerchantSmsDto.builder().requestType("OTP").mobileNumber("7838678139").build();
        SmsDto smsDto =  new SmsDto();
        merchantConfig.setRecipient("rahul.kumar@gmail.com");
        when(notificationMapper.mapSmsDtoToDto(merchantSmsDto)).thenReturn(smsDto);
        notificationDao.sendSmsNotification(merchantSmsDto);
        verify(notificationMapper, times(0)).mapSmsDtoToDto(merchantSmsDto);
        verify(notificationManagementRepository, times(1)).save(any(NotificationManagement.class));
    }



    error

gitSDKToken: null
> Task :bootBuildInfo
> Task :compileJava UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :classes
> Task :compileTestJava
Note: Some input files use unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
> Task :processTestResources NO-SOURCE
> Task :testClasses
Mockito is currently self-attaching to enable the inline-mock-maker. This will no longer work in future releases of the JDK. Please add Mockito as an agent to your build as described in Mockito's documentation: https://javadoc.io/doc/org.mockito/mockito-core/latest/org.mockito/org/mockito/Mockito.html#0.3
WARNING: A Java agent has been loaded dynamically (C:\Users\V1024113\.gradle\caches\modules-2\files-2.1\net.bytebuddy\byte-buddy-agent\1.17.8\f09415827a71be7ed621c7bd02550678f28bc81c\byte-buddy-agent-1.17.8.jar)
WARNING: If a serviceability tool is in use, please run with -XX:+EnableDynamicAgentLoading to hide this warning
WARNING: If a serviceability tool is not in use, please run with -Djdk.instrument.traceUsage for more information
WARNING: Dynamic loading of agents will be disallowed by default in a future release
11:20:40,554 |-INFO in ch.qos.logback.classic.LoggerContext[default] - This is logback-classic version 1.5.21
11:20:40,556 |-INFO in ch.qos.logback.classic.util.ContextInitializer@7cf78c85 - Here is a list of configurators discovered as a service, by rank: 
11:20:40,556 |-INFO in ch.qos.logback.classic.util.ContextInitializer@7cf78c85 -   org.springframework.boot.logging.logback.RootLogLevelConfigurator
11:20:40,556 |-INFO in ch.qos.logback.classic.util.ContextInitializer@7cf78c85 - They will be invoked in order until ExecutionStatus.DO_NOT_INVOKE_NEXT_IF_ANY is returned.
11:20:40,556 |-INFO in ch.qos.logback.classic.util.ContextInitializer@7cf78c85 - Constructed configurator of type class org.springframework.boot.logging.logback.RootLogLevelConfigurator
11:20:40,563 |-INFO in ch.qos.logback.classic.util.ContextInitializer@7cf78c85 - org.springframework.boot.logging.logback.RootLogLevelConfigurator.configure() call lasted 0 milliseconds. ExecutionStatus=INVOKE_NEXT_IF_ANY
11:20:40,563 |-INFO in ch.qos.logback.classic.util.ContextInitializer@7cf78c85 - Trying to configure with ch.qos.logback.classic.util.DefaultJoranConfigurator
11:20:40,564 |-INFO in ch.qos.logback.classic.util.ContextInitializer@7cf78c85 - Constructed configurator of type class ch.qos.logback.classic.util.DefaultJoranConfigurator
11:20:40,567 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback-test.xml]
11:20:40,569 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Found resource [logback.xml] at [file:/D:/Microservices/epay_merchant_service/build/resources/main/logback.xml]
11:20:40,570 |-WARN in ch.qos.logback.classic.util.DefaultJoranConfigurator@ebe067d - Resource [logback.xml] occurs multiple times on the classpath.
11:20:40,570 |-WARN in ch.qos.logback.classic.util.DefaultJoranConfigurator@ebe067d - Resource [logback.xml] occurs at [file:/D:/Microservices/epay_merchant_service/build/resources/main/logback.xml]
11:20:40,570 |-WARN in ch.qos.logback.classic.util.DefaultJoranConfigurator@ebe067d - Resource [logback.xml] occurs at [jar:file:/C:/Users/V1024113/.gradle/caches/modules-2/files-2.1/com.sbi.epay/notification-service/0.1.0/56df5072fb58d6995cec0f34954283fb41633f0d/notification-service-0.1.0.jar!/logback.xml]
11:20:40,570 |-WARN in ch.qos.logback.classic.util.DefaultJoranConfigurator@ebe067d - Resource [logback.xml] occurs at [jar:file:/C:/Users/V1024113/.gradle/caches/modules-2/files-2.1/com.sbi.epay/logging-service/0.1.0/678a2eb9b3364fb6c8a393e64e02d2f5690e6ead/logging-service-0.1.0.jar!/logback.xml]
11:20:40,624 |-WARN in ch.qos.logback.core.joran.action.ConversionRuleAction - [converterClass] attribute is deprecated and replaced by [class]. See element [conversionRule] near line 2
11:20:40,625 |-WARN in ch.qos.logback.core.joran.action.ConversionRuleAction - [converterClass] attribute is deprecated and replaced by [class]. See element [conversionRule] near line 3
11:20:40,625 |-WARN in ch.qos.logback.core.joran.action.ConversionRuleAction - [converterClass] attribute is deprecated and replaced by [class]. See element [conversionRule] near line 4
11:20:40,686 |-INFO in ch.qos.logback.core.model.processor.ConversionRuleModelHandler - registering conversion word methodOrMDC with class [com.sbi.epay.logging.converter.MethodOrMDCConverter]
11:20:40,687 |-INFO in ch.qos.logback.core.model.processor.ConversionRuleModelHandler - registering conversion word classOrMDC with class [com.sbi.epay.logging.converter.ClassOrMDCConverter]
11:20:40,687 |-INFO in ch.qos.logback.core.model.processor.ConversionRuleModelHandler - registering conversion word lineOrMDC with class [com.sbi.epay.logging.converter.LineOrMDCConverter]
11:20:40,687 |-INFO in ch.qos.logback.classic.model.processor.ConfigurationModelHandlerFull - Main configuration file URL: file:/D:/Microservices/epay_merchant_service/build/resources/main/logback.xml
11:20:40,688 |-INFO in ch.qos.logback.classic.model.processor.ConfigurationModelHandlerFull - FileWatchList= {D:\Microservices\epay_merchant_service\build\resources\main\logback.xml}
11:20:40,689 |-INFO in ch.qos.logback.classic.model.processor.ConfigurationModelHandlerFull - URLWatchList= {}
11:20:40,696 |-INFO in ch.qos.logback.core.model.processor.AppenderModelHandler - Processing appender named [CONSOLE]
11:20:40,696 |-INFO in ch.qos.logback.core.model.processor.AppenderModelHandler - About to instantiate appender of type [ch.qos.logback.core.ConsoleAppender]
11:20:40,706 |-INFO in ch.qos.logback.core.model.processor.ImplicitModelHandler - Assuming default type [ch.qos.logback.classic.encoder.PatternLayoutEncoder] for [encoder] property
11:20:40,753 |-INFO in ch.qos.logback.core.ConsoleAppender[CONSOLE] - NOTE: Writing to the console can be slow. Try to avoid logging to the 
11:20:40,753 |-INFO in ch.qos.logback.core.ConsoleAppender[CONSOLE] - console in production environments, especially in high volume systems.
11:20:40,753 |-INFO in ch.qos.logback.core.ConsoleAppender[CONSOLE] - See also https://logback.qos.ch/codes.html#slowConsole
11:20:40,756 |-WARN in ch.qos.logback.core.model.processor.AppenderModelHandler - Appender named [FILE] not referenced. Skipping further processing.
11:20:40,756 |-WARN in ch.qos.logback.core.model.processor.AppenderModelHandler - Appender named [JSON_FILE] not referenced. Skipping further processing.
11:20:40,756 |-WARN in ch.qos.logback.core.model.processor.AppenderModelHandler - Appender named [Logstash] not referenced. Skipping further processing.
11:20:40,756 |-INFO in ch.qos.logback.classic.model.processor.RootLoggerModelHandler - Setting level of ROOT logger to INFO
11:20:40,756 |-INFO in ch.qos.logback.core.model.processor.AppenderRefModelHandler - Attaching appender named [CONSOLE] to Logger[ROOT]
11:20:40,757 |-INFO in ch.qos.logback.core.model.processor.DefaultProcessor@3a4ab7f7 - End of configuration.
11:20:40,758 |-INFO in ch.qos.logback.classic.joran.JoranConfigurator@6badba10 - Registering current configuration as safe fallback point
11:20:40,758 |-INFO in ch.qos.logback.classic.util.ContextInitializer@7cf78c85 - ch.qos.logback.classic.util.DefaultJoranConfigurator.configure() call lasted 194 milliseconds. ExecutionStatus=DO_NOT_INVOKE_NEXT_IF_ANY



notificationMapper.mapSmsDtoToDto(
    MerchantSmsDto(entityId=null, entityType=null, requestType=OTP, mobileNumber=7838678139, message=null)
);
Never wanted here:
-> at com.epay.merchant.dao.NotificationDaoTest.test_SendSms_Notification_success(NotificationDaoTest.java:76)
But invoked here:
-> at com.epay.merchant.dao.NotificationDao.sendSmsNotification(NotificationDao.java:102) with arguments: [MerchantSmsDto(entityId=null, entityType=null, requestType=OTP, mobileNumber=7838678139, message=null)]

org.mockito.exceptions.verification.NeverWantedButInvoked: 
notificationMapper.mapSmsDtoToDto(
    MerchantSmsDto(entityId=null, entityType=null, requestType=OTP, mobileNumber=7838678139, message=null)
);
Never wanted here:
-> at com.epay.merchant.dao.NotificationDaoTest.test_SendSms_Notification_success(NotificationDaoTest.java:76)
But invoked here:
-> at com.epay.merchant.dao.NotificationDao.sendSmsNotification(NotificationDao.java:102) with arguments: [MerchantSmsDto(entityId=null, entityType=null, requestType=OTP, mobileNumber=7838678139, message=null)]

	at com.epay.merchant.dao.NotificationDaoTest.test_SendSms_Notification_success(NotificationDaoTest.java:76)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)


OpenJDK 64-Bit Server VM warning: Sharing is only supported for boot loader classes because bootstrap classpath has been appended

> Task :test
NotificationDaoTest > test_SendSms_Notification_success() FAILED
    org.mockito.exceptions.verification.NeverWantedButInvoked at NotificationDaoTest.java:76
1 test completed, 1 failed
> Task :test FAILED
FAILURE: Build failed with an exception.
* What went wrong:
Execution failed for task ':test'.
> There were failing tests. See the report at: file:///D:/Microservices/epay_merchant_service/build/reports/tests/test/index.html
* Try:
> Run with --scan to get full insights.
BUILD FAILED in 8s
5 actionable tasks: 3 executed, 2 up-to-date

chnageed code  -> 

    public void sendEmailNotification(MerchantEmailDto merchantEmailDto) {
        NotificationManagement notificationManagement = buildNotificationManagement(merchantEmailDto);
        notificationManagementRepository.save(notificationManagement);
        sendEmail(notificationMapper.mapEmailDtoToDto(merchantEmailDto));
        notificationManagement.setStatus(RESPONSE_SUCCESS);
        notificationManagementRepository.save(notificationManagement);
    }

