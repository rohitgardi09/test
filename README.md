“I created this issue, but after analysis, it was found to be related to the network team. Hence, I am closing it.”





9 04:22:11.787 INFO | com.sbi.epay.notification.thirdpartyservice.SmsClient:88 | principal=  | scenario= | operation= | correlation= | lambda$sendSMS$4 | SMS API response.body: null
2026-04-09 04:22:11.787 INFO | com.sbi.epay.notification.thirdpartyservice.SmsClient:89 | principal=  | scenario= | operation= | correlation= | lambda$sendSMS$4 | SMS API full response: <200 OK OK,[Content-Length:"79", Content-Type:"text/plain", Solace-Client-Name:"#rest-ba26223c6d088f1d", Solace-Correlation-ID:"ID:Solace-44819d0fcb37a0f5", Solace-Message-ID:"ID:Solace-44819d0fcb37a0f5"]>
Hibernate: insert into notification_management (content,created_at,entity_id,entity_name,notification_type,request_type,status,updated_at,version,id) values (?,?,?,?,?,?,?,?,?,?)
2026-04-09 04:23:00.037 INFO | com.epay.merchant.schedular.MerchantOnboardingScheduler:42 | principal=  | scenario=MerchantOnboardingScheduler | operation=merchantOnboarding | correlation=13e09bcb-3566-412b-8bdd-36b76c944998 | merchantOnboarding | Find and onboard merchant - Scheduler start time: 1775708580037
2026-04-09 04:23:00.038 INFO | com.epay.merchant.dao.AdminDao:336 | principal=  | scenario=MerchantOnboardingScheduler | operation=merchantOnboarding | correlation=ae9786f2-07bc-43aa-84b4-ac6c4011fc85 | getMerchantConfig | Fetching schedule config for: ONBOARDING
Hibernate: select mce1_0.mc_id,mce1_0.config_key,mce1_0.config_value,mce1_0.created_at,mce1_0.status,mce1_0.updated_at from merchant_config mce1_0 where mce1_0.config_key=? and upper(mce1_0.status)=upper(?)
2026-04-09 04:23:00.046 INFO | com.epay.merchant.dao.AdminDao:338 | principal=  | scenario=MerchantOnboardingScheduler | operation=merchantOnboarding | correlation=ae9786f2-07bc-43aa-84b4-ac6c4011fc85 | getMerchantConfig | Fetched MerchantConfig: Optional[MerchantConfigEntity(mcId=43ff5de5-571f-91c6-e063-0287b10acfe3, configKey=ONBOARDING, configValue=1775475420085, status=ACTIVE)]
2026-04-09 04:23:00.046 INFO | com.epay.merchant.service.AdminService:262 | principal=  | scenario=MerchantOnboardingScheduler | operation=merchantOnboarding | correlation=ae9786f2-07bc-43aa-84b4-ac6c4011fc85 | merchantSyncUp | Calling admin service to get the merchant list for timestamp: 06 Apr 2026 11:37:00 AM
2026-04-09 04:23:00.047 INFO | com.epay.merchant.client.ApiClient:138 | principal=  | scenario=MerchantOnboardingScheduler | operation=merchantOnboarding | correlation=ae9786f2-07bc-43aa-84b4-ac6c4011fc85 | getList | Get call URI: http://admin-adminservice.uat-admin.svc.cluster.local:9094/api/admin/v1/merchant/all/1775475420085
2026-04-09 04:23:00.084 ERROR | com.epay.merchant.client.ApiClient:138 | principal=  | scenario=MerchantOnboardingScheduler | operation=merchantOnboarding | correlation=ae9786f2-07bc-43aa-84b4-ac6c4011fc85 | getList | Unexpected error occurred in scheduled task
com.epay.merchant.exception.MerchantException: 4003 : Data was not found.
	at com.epay.merchant.client.ApiClient.buildResponseList(ApiClient.java:215)
	at com.epay.merchant.client.ApiClient.getList(ApiClient.java:147)
	at com.epay.merchant.externalservice.AdminServicesClient.getMerchants(AdminServicesClient.java:218)
	at com.epay.merchant.dao.AdminDao.getAdminMerchants(AdminDao.java:316)
	at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:103)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:355)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:716)
	at com.epay.merchant.dao.AdminDao$$SpringCGLIB$$0.getAdminMerchants(<generated>)
	at com.epay.merchant.service.AdminService.merchantSyncUp(AdminService.java:263)
	at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:103)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:355)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:196)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:768)
	at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:379)
	at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:119)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:184)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:768)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:720)
	at com.epay.merchant.service.AdminService$$SpringCGLIB$$0.merchantSyncUp(<generated>)
	at com.epay.merchant.schedular.MerchantOnboardingScheduler.merchantOnboarding(MerchantOnboardingScheduler.java:46)
	at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:103)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:355)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:196)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:768)
	at net.javacrumbs.shedlock.core.DefaultLockingTaskExecutor.executeWithLock(DefaultLockingTaskExecutor.java:72)
	at net.javacrumbs.shedlock.spring.aop.MethodProxyScheduledLockAdvisor$LockingInterceptor.invoke(MethodProxyScheduledLockAdvisor.java:83)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:184)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:768)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:720)
	at com.epay.merchant.schedular.MerchantOnboardingScheduler$$SpringCGLIB$$0.merchantOnboarding(<generated>)
	at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:103)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	at org.springframework.scheduling.support.ScheduledMethodRunnable.runInternal(ScheduledMethodRunnable.java:130)
	at org.springframework.scheduling.support.ScheduledMethodRunnable.lambda$run$2(ScheduledMethodRunnable.java:124)
	at io.micrometer.observation.Observation.observe(Observation.java:499)
	at org.springframework.scheduling.support.ScheduledMethodRunnable.run(ScheduledMethodRunnable.java:124)
	at org.springframework.scheduling.support.DelegatingErrorHandlingRunnable.run(DelegatingErrorHandlingRunnable.java:54)
	at org.springframework.scheduling.concurrent.ReschedulingRunnable.run(ReschedulingRunnable.java:96)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:572)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:317)
	at java.base/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:304)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1144)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:642)
	at java.base/java.lang.Thread.run(Thread.java:1583)
