Subject: Follow-up on OpenShift Access
Hi Team,
Just following up on my OpenShift access request. Please provide access to the DEV, UAT, and PRE-PROD environments.
Thanks & Regards,
Rohit Gardi






Subject: Please Add application-local.yml
Hi Team,
application-local.yml is not added. Because of this, whenever we need to test or fix anything in the local environment, we have to add all the required configurations manually first.
Please add application-local.yml.







Required Backlog Changes – Please Retain



Hi Team,

These are required backlog changes, so please retain them. If any additional changes are required, please add them on top of these changes instead of reverting the MR.

Thanks,
Rohit


Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'paymentInitiationService' defined in file [E:\Java_Backend\epay_transaction_service\build\classes\java\main\com\epay\transaction\service\PaymentInitiationService.class]: Unsatisfied dependency expressed through constructor parameter 0: Error creating bean with name 'paymentInitiationDao' defined in file [E:\Java_Backend\epay_transaction_service\build\classes\java\main\com\epay\transaction\dao\PaymentInitiationDao.class]: Unsatisfied dependency expressed through constructor parameter 5: Error creating bean with name 'paymentDao' defined in file [E:\Java_Backend\epay_transaction_service\build\classes\java\main\com\epay\transaction\dao\PaymentDao.class]: Unsatisfied dependency expressed through constructor parameter 0: Error creating bean with name 'constructPaymentServicesClient': Injection of autowired dependencies failed

file -> application local.yml he add kela ahe mi 

payment.otherinb.gtwmapid: 57

