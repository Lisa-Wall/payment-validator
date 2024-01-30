# payment-validator
Validates a payment against an expected payment.

This is the glue that joins two other modules: interac-parser and receipt-generator.

# Purpose

This component is part of a group of components that work together to produce rent receipts automatically in response to a client sending an eTransfer.

Components:

1. interac-parser: converts an Interac email into a JSON object containing the data relevant to the receipt.
2. payment-validator: validates an Interac payment (JSON object from #1) and produces another JSON object formatted ready to for PDF conversion.
3. receipt-generator (this component): Converts the JSON object from #2 to a PDF receipt
4. receipt-mailer: Mails the receipt to the client.

This project is component #2 in the above list, the payment validator.

# Input

A JSON object representing the Interac data. Note that this input is the output from the interac-parser module from step 1 above.

Sample input:

```JavaScript

{
  version: '2.0',
  uid: undefined,
  serverId: undefined,
  messageId: '<1569034797.175709886.1598538700364.JavaMail.app_prod@mtlpnot04.prod.certapay.com>',
  date: 2024-01-05T14:31:40.000Z,
  toName: 'SAMPLE LANDLORD',
  toEmail: 'addmoney@landlord_company.ca',
  fromName: 'SAMPLE TENANT',
  fromEmail: 'catch@payments.interac.ca',
  replyToName: 'SAMPLE TENANT',
  replyToEmail: 'sample.tenant@gmail.com',
  language: 'en',
  amount: 5.25,
  currency: 'CAD',
  reference: 'CAvjMTys',
  isAutoDeposit: true,
  userMessage: '((UON5-CHCZ-22RH-F0N6))',
  isEmailTransfer: true,
  body: 'Hi SAMPLE LANDLORD,\n' +
    'SAMPLE TENANT has sent you a money transfer for the amount of $5.25 (CAD) and the money has been automatically deposited into your bank account at Scotiabank.\n' +
    'Message: ((UON5-CHCZ-22RH-F0N6))\n' +
    'Reference Number : CAvjMTys\n' +
    'Please do not reply to this email.\n' +
    'Frequently Asked Questions: http://www.interac.ca/consumers/faqs.php#emt\n' +
    'This email was sent to you by Interac Corp., the owner of the INTERAC e-Transfer®\n' +
    'service, on behalf of Scotiabank.\n' +
    'Interac Corp.\n' +
    'Royal Bank Plaza, North Tower, 200 Bay Street, Suite 2400\n' +
    'P.O. Box 45, Toronto, ON M5J 2J1\n' +
    'www.interac.ca\n' +
    '® Trade-mark of Interac Corp. Used under license.',
  subject: 'INTERAC e-Transfer: A money transfer from SAMPLE TENANT has been automatically deposited.',
  errors: null
}
```

## Output

If validation is successful: An object containing the data required to generate a receipt.

If validation fails: Steps should be taken to notify interested parties. Interested parties could be generalized to be whoever has subscribed to notifications for this, but at this point, we just have ourselves to notify so we'll probably just keep it simple and send ourselves an email. The sender (tenant) should be told too, possibly with next steps/instructions. Perhaps the payment amount was incorrect, for example, and they need to send a further payment.

Successful validation should result in an object such as this:

```
{
    logoFilePath: "",                      // Optional. Logo should be approx 80px square.
    receiptTitle: "Receipt",               // ex: "Rent Receipt", "Service Receipt"
    dateOfIssue: "2024-01-15",             // String. ex: new Date().toLocaleDateString()
    amount: "745.00",                      // String representation of the amount paid.
    receivedFrom: "Sample Tenant",
    receivedBy_Name: "Sample Landlord",
    receivedBy_Title: "",                  // Optional. ex: (landlord)
    receivedBy_signatureImageFilePath: "", // Optional. If you want a signature on the receipt.
    paymentMethod: "",                     // ex: "eTransfer", "Cheque", "Cash", "Sweat Equity"
    rentalAddress: "1-100 Tenant Avenue, Ottawa ON, K1P 5S3",
    month: "January, 2024"
}
```

## Further Considerations

I wonder whether to store anything in a database. I'm trying to keep overhead minimal. But if a payment comes in part way through a month, how do I know if it's late rent for the past month or early rent for the next month, without some kind of rent roll? Maybe design something ultra simple, with a notes field to explain anything unusual. Something like this:
  ```
  Payment
  {
    name: "Sample Tenant",
    amount: "745.00",
    month: "02-2024",
    full_or_partial: "partial",
    late_fees: "4.00",
    path_PDF: "",
    notes: "Rent was paid in two payments and this is the second and final payment."
  }
 
