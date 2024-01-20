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

More details to follow.
