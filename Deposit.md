# Deposit Definition

| Parameter      | Type    | Description                                           |
|----------------|---------|-------------------------------------------------------|
| amount         | number  | Deposit amount                                        |
| currency       | string  | Deposit currency                                      |
| transactions       | array  | Array of [transactions](#transaction-definition)                                      |

### Transaction Definition 
| Parameter      | Type    | Description                                           |
|----------------|---------|-------------------------------------------------------|
| id         | number  | Transaction identifier                                        |
| amount         | number  | Transaction amount                                        |
| currency       | string  | Transaction currency                                      |
| paidAtUtc   | array   | Date and time  in UTCwhen this transaction was paid |
