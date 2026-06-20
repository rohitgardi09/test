
{
  "operatingMode": "ONLINE",
  "payProcId": "PP001",
  "payProcType": "CARD",
  "gtwMapsId": "GTW001",
  "pgBankCode": "SBI",
  "merchPostedAmount": 1000.00,
  "transactionAmount": 1000.00,
  "upiAddress": "user@upi",
  "channelBank": "SBI",
  "altNumber": "9876543210",
  "expiryMonth": "12",
  "expiryYear": "2028",
  "cvv": "123",
  "cardHolderName": "Ranjan Kumar",
  "customerGSTNDetails": {
    "gstn": "27ABCDE1234F1Z5",
    "legalName": "ABC Traders Pvt Ltd",
    "email": "abc@example.com",
    "mobileNum": "9876543210"
  }
}



scheduled:
  report:
    upload:
      retry: "0 */5 * * * *"
