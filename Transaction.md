# Transaction Definition

| Parameter      | Type    | Description                                           |
|----------------|---------|-------------------------------------------------------|
| id | number | Unique identifier for this transaction | 
| referenceCode  | string | Transaction reference code                                 |
| confirmedAtUtc   | date   | Date and time  in UTC when this transaction was confirmed. This is the moment when all payments are confirmed and accredited. |
| totalPrice         | number  | Total price for this transaction, including all quantities purchased, fees and tips. |
| totalFees         | number  | Total amount corresponding to fees for this transaction. |
| totalTips         | number  | Total amount corresponding to tips for this transaction. |
| totalPaid         | number  | Total amount paid by customer. |
| currency       | string  | Currency in which the transaction amounts are specified. |
| paxs | number | Total number of attendees for this transaction. |
| products | Array of [Products](./Product.md) | Products included in this transaction, such as tickets, prepaid reservations, extras, etc |
