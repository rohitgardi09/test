Dear Team,

We have initiated work on the Consent Management implementation. Prior to proceeding further, we have made certain assumptions for the development. Please find the details below:

1. We have assumed the use of touchPointId and purposeSetId for this implementation.
2. In case the EIS service is unavailable and we are unable to record user consent, we will continue prompting the user for consent until the EIS service becomes operational. During this period, the consent will be temporarily stored in CCMS.
3. For retrying the consent write operation to CCMS, we have assumed a maximum of three attempts.
4. As we have not yet received the sample request and response from the EIS team, we are currently using dummy request and response data provided by the product team in the Product Note document. We will align with the official format once it is shared.

Please note that the request and response structure we intend to use for EIS has been attached to this email for your reference.

Kindly review and let us know if any changes are required.

Thanks and regards,
Saurabh Mahto





* - Create NB DV Endpoint with Transaction Token

We need to create dv endpoint used by front end payment page to know current nb transaction status before updating a transaction to cancel. 
this endpoint will call payment service to know the status of the transaction.

front end point will call the transaction service end point, Transaction service will provide the response to front end.
Transaction service can call payment service to know the exact status at channel end (double verification call of channel bank) 
and based on double verification status transaction will mark Success/Failed by payment service.
Cancel status would be in description of failure reason "Cancel by user & channel bank response- {response message from channel bank}"
