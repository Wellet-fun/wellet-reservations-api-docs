# Wellet Reservations API Reference
This repository contains documentation for using the [Wellet](https://wellet.us/) Reservations API for obtaining information and updating reservations made through the Wellet platform.

This api is organized around [REST](https://en.wikipedia.org/wiki/REST), accepts [JSON](https://www.json.org/json-en.html) request bodies, returns [JSON-encoded](https://www.json.org/json-en.html) responses, and uses standard HTTP response codes and verbs.


## Environments
Wellet provides a production environment and a sandbox environment. The sandbox environment allows developers to integrate with the Wellet platform without affecting critical data. All the reservations functionality is fully implemented in both environments. Sandbox is important for testing the functionality and logic of third party applications before deploying to production or releasing to customers.

## Base URLs
The Reservations API is served over HTTPs. All URLs referenced in the documentation have the following base:

| Environment |          Base URL           |
|:-----------:|:---------------------------:|
| Production  | https://wr-api.wellet.cloud |
| Sandbox     | https://wr-api.wellet.dev   |

## Authentication

All the calls to this api need to specify an api key in the `x-api-key` http header. The api key will be provided by the Wellet team. Contact [info@wellet.fun](info@wellet.fun) if you want to integrate with the Wellet platform.

## Endpoints

## Get Venues
Returns a list of all the active venues. A venue represents a place that can be reserved, such as a restaurant or a bar.

```
GET /venues
```

#### Input Parameters
None

#### Output Parameters
Returns an array of venue, each venue containing the following properties:

| Parameter | Type    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| id        | string  | Venue identifier code                                  |
| name      | string  | Name of the venue                                      |

#### Example

Example request:
```bash
curl --location --request GET 'https://wr-api.wellet.dev/venues/' \
--header 'x-api-key: YOUR_API_KEY'
```

Example response:
```json
[
    {
        "id": "acme",
        "name": "Acme Resto",
    },
    {
        "id": "bites",
        "name": "Bites",
    },
    {
        "id": "fuego",
        "name": "Fuego Resto",
    }
]
```
## Get Venue Reservations
Returns the list of reservations confirmed for a particular venue and a particular date. Note: reservations that have reached the maximum number of payments are not returned, as they are not valid anymore to record payments.

```
GET /venues/{venueId}/reservations
```
#### Input Parameters
| Parameter | Location     | Type    | Description                                                  |
|-----------|--------------|---------|--------------------------------------------------------------|
| venueId   | Path         | string  | Venue identifier                                             |
| date      | Query String | string  | Reservation date. Format: "yyyy-mm-dd". Optional: if omitted, it considers the current date in local time. |

#### Output Parameters
Returns an array of reservations, each of them with the following properties:

| Parameter | Type    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| code      | string  | Reservation code                                       |
| date      | string  | Reservation date in "yyyy-mm-dd" format.               |
| time      | string  | Reservation time in "hh:mm" 24-hour notation format.   |
| paxs      | int     | Number of people in the group.                         |
| customerName | string | Full name of the customer associated with the reservation |
| conciergeName | string | Full name of the concierge that generated the reservation |

#### Example

Example request:
```bash
curl --location --request GET 'https://wr-api.wellet.dev/venues/acme/reservations?date=2024-01-04' \
--header 'x-api-key: YOUR_API_KEY'
```

Example response:
```json
[
    {
        "code": "TSRH",
        "date": "2024-01-04",
        "time": "20:00",
        "paxs": 4,
        "customerName": "Laura Garcia",
        "conciergeName": "Diego Sánchez"
    },
    {
        "code": "GFAL",
        "date": "2024-01-04",
        "time": "20:30",
        "paxs": 8,
        "customerName": "Eduardo Lopez",
        "conciergeName": "Valeria Pérez"
    }
]
```

## Get Reservation by code
Returns a particular reservation by its code. This endpoint can be used to validate if a particular reservation code is valid.

```
GET /venues/{venueId}/reservations/{reservationCode}
```

#### Input Parameters
| Parameter | Location     | Type    | Description                                                  |
|-----------|--------------|---------|--------------------------------------------------------------|
| venueId   | Path         | string  | Venue identifier                                             |
| reservationCode | Path | string  | Reservation code. |

#### Output Parameters
If the reservation is found and it is confirmed, returns a reservation with the following properties:

| Parameter | Type    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| code      | string  | Reservation code                                       |
| date      | string  | Reservation date in "yyyy-mm-dd" format.               |
| time      | string  | Reservation time in "hh:mm" 24-hour notation format.   |
| paxs      | int     | Number of people in the group.                         |
| customerName | string | Full name of the customer associated with the reservation |
| conciergeName | string | Name of the concierge that generated the reservation |

#### Example

Example request:
```bash
curl --location --request GET 'https://wr-api.wellet.dev/venues/acme/reservations/GFAL' \
--header 'x-api-key: YOUR_API_KEY'
```

Example response:
```json
{
    "code": "GFAL",
    "date": "2024-01-04",
    "time": "20:30",
    "paxs": 8,
    "customerName": "Eduardo Lopez",
    "conciergeName": "Valeria Pérez"
}
```

#### Error Codes
The following HTTP Status Codes can be returned by this endpoint:

| Code      | Status    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| 400       | Bad Request | The reservation was found but it is not valid for receiving payments. One of the following error codes are returned in the response: <br>- `MAX_PAYMENTS_REACHED`: The number of payments received for this reservation has reached its limit.|
| 404       | Not Found | The reservation was not found or it is  not confirmed (for example: it has been cancelled). The error description returned is `RESERVATION_NOT_FOUND`|

Example error responses:

```json
{
    "code": 400,
    "errorDescription": "MAX_PAYMENTS_REACHED"
}
```

```json
{
    "code": 403,
    "errorDescription": "RESERVATION_NOT_FOUND"
}
```

## Register a payment for a reservation
Registers a new payment for a particular reservation. If this endpoint is called multiple times with differnt payment ids for the same reservation code, multiple payments will be added to the reservation. Note: if two payments are registered with the same id for the same reservation, only the first one will be registered, as the api assumes that this is a duplicated record sent by accident.

```
PUT /venues/{venueId}/reservations/{reservationCode}/payment
```

#### Input Parameters
| Parameter | Location     | Type    | Description                                                  |
|-----------|--------------|---------|--------------------------------------------------------------|
| venueId   | Path         | string  | Venue identifier                                             |
| reservationCode | Path | string  | Reservation code. |
| paymentId      | Body | string  | Any string identifying the payment (optional). |
| amount      | Body | number  | Commissionable amount for this check. |
| currency      | Body | string  | Currency of payment. |
| total      | Body | number  | The final amount the guest needs to pay, which includes the subtotal, tax, and any service charges (optional). |
| subtotal      | Body | number  | The total cost of all items ordered before any additional charges, like tax or service fees (optional). |
| tax      | Body | number  | The government-imposed tax on the bill amount, usually a percentage of the subtotal (optional). |
| serviceCharge      | Body | number  | An optional fee added by the venue, often used as a tip or to cover staffing costs (optional). |
| tableNumber | Body | string | Table number or identifier, as it is displayed to the final user.|
| paxs | Body | number | Number of people in the group.|
| openedAt | Body | date | Timestamp when this particular table was opened in the system. Format: `yyyy-MM-ddTHH:mm:ss` (ISO 8601 format in local time)|
| hasDiscount | Body | boolean | True if a discount was applied to this table. (Optional, false by default) |
| discountName | Body | string | Name of the discount applied (optional). |
| discountAmount | Body | number | The amount of the discount applied, if any. (Optional, default: 0)|
| checkNumber | Body | string | The check nunber of the table. (optional)|
| checkUrl | Body | string | Check url containing a PDF file or image of the check (optional)|
| events | Body | [events](./Events.md) | Date and user information about events detailed [here](./Events.md). (Optional)|
| reservationDate | Body | string | Reservation date. Format: "yyyy-mm-dd". Optional, only needed when reservation is closed and it is needed to patch payments not registered for some reason.|
| items | Body | Array of [items](./CheckItem.md) | Collection of products ordered along with its quantity and price. |


#### Output Parameters
If the payment was successfully registered, the following parameters are returned:

| Parameter | Type    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| code      | string  | Reservation code                                       |
| date      | string  | Reservation date in "yyyy-MM-dd" format.               |
| time      | string  | Reservation time in "HH:mm" 24-hour notation format.   |
| paxs      | int     | Number of people in the group.                         |
| customerName | string | Full name of the customer associated with the reservation |
| conciergeName | string | Name of the concierge that generated the reservation |
| totalPaidAmount | number | Total amount paid for this order (sum of all payments received) |
| currency      | string  | Currency of payments. |
| payments  | array | Array of [payment](./Payment.md) objects|

#### Error Codes
The following HTTP Status Codes can be returned by this endpoint:

| Code      | Status    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| 200       | OK      | The payment was successfully registered. |
| 400       | Bad Request | The reservation was found but it is not valid for receiving payments. One of the following error codes are returned in the response: <br>- `MAX_PAYMENTS_REACHED`: The number of payments received for this reservation has reached its limit.|
| 404       | Not Found | The reservation was not found or it is  not confirmed (for example: it has been cancelled).|

#### Example

Example request:
```bash
curl --location --request PUT 'https://wr-api.wellet.dev/venues/acme/reservations/GFAL/payment' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "paymentId": "ABC123",
    "amount": 15383.25,
    "currency": "MXN",
    "tableNumber": "23",
    "paxs": 8,
    "openedAt": "2024-01-04T13:23:58",
    "hasDiscount": true,
    "discountName": "Locales",
    "discountAmount": 1709.25,
    "checkNumber": "205",
    "checkUrl": "https://billing-xyz.com/checks/check205.pdf",
    "reservationDate": "2024-01-04",
    "events": {
        "tableOpened": {
            "date": "2024-01-04T13:23:58",
            "userId": "a38e8b20-7299-4b5b-b6b6-cb86a472fd9f",
            "userFullName": "Alejandra Ramírez"
        },
        "welletCodeAdded": {
            "date": "2024-01-04T13:28:12",
            "userId": "f4a6e7d3-8211-4e0d-bd3c-ee7f10706c87",
            "userFullName": "Carlos Rodríguez"
        },
        "tableClosed": {
            "date": "2024-01-04T15:33:02",
            "userId": "7f64c20d-9cb1-4a7e-8c84-91e96f4240a3",
            "userFullName": "Luis Gonzalez"
        },
        "tablePaid": {
            "date": "2024-01-04T15:35:19",
            "userId": "7f64c20d-9cb1-4a7e-8c84-91e96f4240a3",
            "userFullName": "Luis Gonzalez"
        }
    },
    items: [
        {
            "name": "Hokkien Street Noodles",
            "quantity": 8,
            "price": 2400
        },
        {
            "name": "Crispy Honey Shrimp",
            "quantity": 3,
            "price": 2080.00
        },
        {
            "name": "Wok-Charred Brussel Sprouts",
            "quantity": 3,
            "price": 1800.60
        },
        {
            "name": "Mojito",
            "quantity": 8,
            "price": 2400
        },
        {
            "name": "Valhalla Reserve Cabernet",
            "quantity": 3,
            "price": 3200
        },
        {
            "name": "Gold Sparkle Brut",
            "quantity": 4,
            "price": 3502.65
        }
    ]
}'
```

Example response:
```json
{
    "code": "GFAL",
    "date": "2024-01-04",
    "time": "20:30",
    "paxs": 8,
    "customerName": "Eduardo Lopez",
    "conciergeName": "Valeria Pérez",
    "totalPaidAmount": 20383.25,
    "currency": "MXN",
    "payments": [
        {
            "id": "ABC122",
            "amount": 5000.00,
            "currency": "MXN",
            "createdAtUtc": "2024-01-04T21:01:53.31"
        },
        {
            "id": "ABC123",
            "amount": 15383.25,
            "currency": "MXN",
            "createdAtUtc": "2024-01-05T21:05:03.3706244Z"
        }
    ]
}
```

# Venue Events
The `Event` object represents any scheduled activity or occasion at a specific venue where tickets or vip areas are available for purchase.

## Get Events
Retrieves a list of all events scheduled within a specified date range, including events that have already occurred in the past.

```
GET /venues/{venueId}/events?startDate={startDate}&endDate={endDate}
```

#### Input Parameters
| Parameter | Location     | Type    | Description                                                  |
|-----------|--------------|---------|--------------------------------------------------------------|
| venueId   | Path         | string  | Venue identifier                                             |
| startDate | Query String | string  | The start date (in the venue's local time) of the range in ISO 8601 format (e.g., 2025-12-22). Only events starting on or after this date will be included.|
| endDate | Query String | string  | The end date (in the venue's local time) of the range in ISO 8601 format (e.g., 2025-12-31). Only events ending on or before this date will be included.|

#### Output Parameters
Returns an array of events, each of them with the following properties:

| Parameter | Type    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| id      | string  | A unique identifier for the event.                        |
| startDateTime      | date  | The local date and time when the event begins. Format: `yyyy-MM-ddTHH:mm:ss` (ISO 8601 format in local time). |
| endDateTime      | date  | The local date and time when the event ends. Format: `yyyy-MM-ddTHH:mm:ss` (ISO 8601 format in local time). |
| name      | string     | The name of the event, as displayed to users.  |
| venueId     | string  | Venue identifier. |

#### Example

Example request:
```bash
curl --location --request GET 'https://wr-api.wellet.dev/venues/acme/events?startDate=2025-01-10&endDate=2025-01-17' \
--header 'x-api-key: YOUR_API_KEY'
```

Example response:
```json
[
    {
        "id": "infinite-vibes",
        "startDateTime": "2025-01-13T22:00:00",
        "endDateTime": "2025-01-14T02:00:00",
        "name": "Infinite Vibes",
        "venueId": "acme"
    },
    {
        "id": "feel-good-fridays",
        "startDateTime": "2025-01-16T22:00:00",
        "endDateTime": "2025-01-17T02:00:00",
        "name": "Feel Good Fridays",
        "venueId": "acme"
    } 
]
```

# Transaction
Represents a transaction made for a specific event, such as the purchase of a ticket, a VIP table reservation, a cocktail package, etc.

## Get list of transactions for an event
Returns a list of transactions made for a specific event.
```
GET /venues/{venueId}/events/{eventId}/transactions
```
#### Input Parameters
| Parameter | Location     | Type    | Description                                                  |
|-----------|--------------|---------|--------------------------------------------------------------|
| venueId   | Path         | string  | Venue identifier                                             |
| eventId | Path | string  | Event identifier. |

#### Output Parameters
| Parameter | Type    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| totalTransactions      | number  | Number of transactions for this event.                        |
| totalPaid      | number  | The sum of all the amounts paid in the transactions for this event. |
| currency      | string  | Currency for `totalPaid`. |
| transactions      | Array of [Transaction](./Transaction.md)     | Array of all the transactions for this event.  |

#### Example

Example request:
```bash
curl --location --request GET 'https://wr-api.wellet.dev/venues/acme/events/infinite-vibes/purchases' \
--header 'x-api-key: YOUR_API_KEY'
```

Example response:
```json
{
    "totalTransactions": 2,
    "totalPaid": 800.00,
    "currency": "EUR",
    "transactions": [
        {
            "id": 45678,
            "referenceCode": "TGJM",
            "confirmedAtUtc": "2025-01-12T14:32:12",
            "totalPrice": 300,
            "totalFees": 36,
            "totalTips": 30,
            "totalPaid": 300,
            "currency": "EUR",
            "paxs": 4,
            "products": [
                {
                    "id": 2358,
                    "name": "VIP Ticket",
                    "quantity": 4,
                    "totalPrice": 300,
                    "totalFees": 36,
                    "totalTips": 30
                }
            ]
        },
        {
            "id": 48578,
            "referenceCode": "FRDE",
            "confirmedAtUtc": "2025-01-13T16:21:44",
            "totalPrice": 500,
            "totalFees": 60,
            "totalTips": 15,
            "totalPaid": 500,
            "currency": "EUR",
            "paxs": 4,
            "products": [
                {
                    "id": 2358,
                    "name": "VIP Ticket",
                    "quantity": 2,
                    "totalPrice": 150,
                    "totalFees": 18,
                    "totalTips": 15
                },
                {
                    "id": 1348,
                    "name": "Coctel personalizado",
                    "quantity": 2,
                    "totalPrice": 350,
                    "totalFees": 42,
                    "totalTips": 0
                }
            ]
        },
    ]
}
```

## Get list of transactions for a venue
Returns all transactions for a particular venue filtered by creation date or by `afterTransactionId`, ordered by date of creation or update. 

#### Input Parameters
| Parameter | Location     | Type    | Description                                                  |
|-----------|--------------|---------|--------------------------------------------------------------|
| venueId   | Path         | string  | Venue identifier                                             |
| fromDateUtc| Query String | date | Minimum transaction date in UTC. Format: "yyyy-mm-dd". |
| toDateUtc| Query String | date | (optional) Maximum transaction date in UTC. Format: "yyyy-mm-dd". |
| afterTransactionId | Path | string  | (optional) The unique identifier of the last processed transaction. Used to retrieve all transactions that occurred after this specific transaction for synchronization purposes. |

#### Output Parameters
Array of [Transactions](./Transaction.md) object with its corresponding [Event](#venue-events).

#### Example

Example request:
```bash
curl --location --request GET 'https://wr-api.wellet.dev/venues/acme/transactions?fromDateUtc=2025-01-01&afterTransactionId=45678' \
--header 'x-api-key: YOUR_API_KEY'
```

Example response:
```json
[
    {
        "id": 48578,
        "referenceCode": "FRDE",
        "confirmedAtUtc": "2025-01-12T14:32:12",
        "totalPrice": 300,
        "totalFees": 36,
        "totalTips": 30,
        "totalPaid": 300,
        "currency": "EUR",
        "paxs": 4,
        "products": [
            {
                "id": 2358,
                "name": "VIP Ticket",
                "quantity": 4,
                "totalPrice": 300,
                "totalFees": 36,
                "totalTips": 30
            }
        ],
        "event": {
            "id": "infinite-vibes",
            "startDateTime": "2025-01-13T22:00:00",
            "endDateTime": "2025-01-14T02:00:00",
            "name": "Infinite Vibes",
        }
    } 
]
```


# Webhooks

Webhooks serve as communication mechanism **from the Wellet platform to your platform**. They allow Wellet to automatically send real-time notifications and event data to your system whenever specific actions occur—such as opening a table or processing a payment. This push-based communication ensures that your platform stays synchronized with events happening in Wellet, without the need for manual polling or intervention.

In contrast, **API endpoints operate in the opposite direction**: they are called by your platform to send data or request information from the Wellet platform.

## Enabling Webhooks

The following information is required for enabling Webhooks:

* The webhook URL where you would like POST request payloads sent.
* Custom request headers you would like sent with request payloads as key-value pairs in the following format
Example: "Authorization:AuthToken##OtherHeader:HelloWorld"

Contact the Wellet team if you need to enable webhooks in your platform.

## Event overview
Wellet generates event data that we can send you to inform you of activity related to Wellet reservations. Each event payload sent to webhooks has the following format:

```javascript
{
    "id": "0d9ef8e9-bb69-466e-abc6-c18e413de270",
    "timestamp": "2024-10-29T19:56:55.7544784Z",
    "apiVersion": "1.0",
    "object": "event",
    "type": "EVENT TYPE",
    "data": {
        // specific payload for this event type
    } 
}
```

## Event types

### `reservation.arrived` - Reservation Arrived
This webhook event notifies that the guests of a reservation have arrived and are ready to be seated. This is particularly useful for integrating a POS system with Wellet, allowing the POS to automatically open the table and assign the corresponding reservation code, eliminating the need for manual intervention.

### Payload description
| Parameter | Type    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| code      | string  | Reservation code. This code needs to be used when registering payments for this reservation ([endpoint](#register-a-payment-for-a-reservation)). |
| venueId     | string  | Venue identifier. |
| table     | string  | Table number where guests are or will be seated. Null if booking engine does not provide this information. |
| customerName | string | Customer name associated with the reservation. |
| paxs | number | Number of people in the group for this reservation. |
| prepayment | [Prepayment](./Prepayment.md)  | Prepayment information for this reservation. |
| discounts  | array of [discounts](./Discount.md) | Array of discounts for applying promotions, loyalty discounts, or special offers directly to the bill. Empty array if none. |


### Payload example:

```javascript
{
    "id": "0d9ef8e9-bb69-466e-abc6-c18e413de270",
    "timestamp": "2024-10-29T19:56:55.7544784Z",
    "apiVersion": "1.0",
    "object": "event",
    "type": "reservation.arrived",
    "data": {
        "code": "GFAL",
        "venueId": "acme",
        "table": "12",
        "customerName": "Eduardo Lopez",
        "paxs": 4,
        "prepayment": {
            "total": 9500.23,
            "currency": "MXN",
            "credit": 4500,
            "accessFee": 4000.23,
            "products": [
                {
                    "id": "SER025",                    
                    "locationId": "Palapa",
                    "total": 9500.23,
                    "credit": 4500,
                    "accessFee": 4000.23,
                    "paxs": 4
                }
            ],
            "payments": [
                {
                    "id": "chrg-343523",
                    "amount": 9500.23,
                    "currency": "MXN",
                    "createdAtUtc": "2024-10-22T14:32:12"                   
                }
            ]
        }
        "discounts": [
            {
                "name": "Premium Customer", 
                "percentageDiscount": 8.5
            }
        ]
    } 
}
```

### `transaction.confirmed` - Transaction Confirmed
This webhook event notifies that a transaction for purchasing products has been confirmed, which means that all the payments have been confirmed and accredited.

### Payload description
A [Transaction](./Transaction.md) object with its corresponding [Event](#venue-events).


### Payload example:

```javascript
{
    "id": "0d9ef8e9-bb69-466e-abc6-c18e413de270",
    "timestamp": "2025-01-12T14:32:12",
    "apiVersion": "1.0",
    "object": "event",
    "type": "transaction.confirmed",
    "data": {
        "id": 45678,
        "referenceCode": "TGJM",
        "confirmedAtUtc": "2025-01-12T14:32:12",
        "totalPrice": 300,
        "totalFees": 36,
        "totalTips": 30,
        "totalPaid": 300,
        "currency": "EUR",
        "paxs": 4,
        "products": [
            {
                "id": 2358,
                "name": "VIP Ticket",
                "quantity": 4,
                "totalPrice": 300,
                "totalFees": 36,
                "totalTips": 30
            }
        ],
        "event": {
           "id": "infinite-vibes",
            "startDateTime": "2025-01-13T22:00:00",
            "endDateTime": "2025-01-14T02:00:00",
            "name": "Infinite Vibes",
        }
    } 
}
```



