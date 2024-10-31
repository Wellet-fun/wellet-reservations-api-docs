# Prepayment Definition

Represents a deposit prepaid by a customer to secure the reservation.

| Parameter      | Type    | Description                                           |
|----------------|---------|-------------------------------------------------------|
| total         | number  | Total amount prepaid by customer                     |
| currency       | string  | Payment currency                                      |
| credit       | number  | The portion of the prepaid amount that serves as a credit toward the final bill. |
| accessFee   | number   | The portion of the prepaid amount charged for access to the specific reserved area. |
| products | Array of [products](#product) | Array of prepaid products |
| payments | Array of [payments](./Payment.md) | Detail of each payment made by customer for securing this reservation. |

# Product

| Parameter      | Type    | Description                                           |
|----------------|---------|-------------------------------------------------------|
| id             | string | Product identifier                                    |
| total         | number  | Total amount prepaid for this product                     |
| credit       | number  | The portion of the prepaid amount that serves as a credit toward the final bill. |
| accessFee   | number   | The portion of the prepaid amount charged for access to this product. |
| paxs | number | Number of people in the group for this product. |


