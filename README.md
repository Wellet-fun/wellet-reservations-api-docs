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
# POS Communication
Wellet requires the payment amount for each reservation made on its platform in order to calculate and allocate commissions. To achieve this, communication with the POS system responsible for processing payments is necessary.

The mapping between Wellet reservations and POS transactions is facilitated through one of the following mechanisms:

* By [Reservation Code](#by-reservation-code)
* By [Table Id](#by-table-id)

Mapping via table ID is only applicable when table assignments are managed within the Wellet platform.

# By Reservation Code

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
    "code": 404,
    "errorDescription": "RESERVATION_NOT_FOUND"
}
```

## Register a payment for a reservation by reservation code
Registers a new payment for a particular reservation. If this endpoint is called multiple times with different payment ids for the same reservation code, multiple payments will be added to the reservation. Note: if two payments are registered with the same id for the same reservation, only the first one will be registered, as the api assumes that this is a duplicated record sent by accident.

```
PUT /venues/{venueId}/reservations/{reservationCode}/payment
```

#### Input Parameters
| Parameter | Location     | Type    | Description                                                  |
|-----------|--------------|---------|--------------------------------------------------------------|
| venueId   | Path         | string  | Venue identifier                                             |
| reservationCode | Path | string  | Reservation code. |
| paymentId      | Body | string  | Any string identifying the payment (optional). |
| amount      | Body | number  | Amount paid. |
| currency      | Body | string  | Currency of payment. Possible values: 'MXN' |
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


# By Table Id
It is essential to ensure that table IDs are created in the Wellet platform exactly as they are recorded in the POS system. Failure to do so will result in the mapping not functioning correctly.

## Open a table from POS
Notifies Wellet that a table has been opened in the POS system, allowing Wellet to begin recording payments for the reservation. It is crucial that the table is assigned in Wellet prior to it being opened in the POS.

```
POST /venues/{venueId}/table/{tableId}/open
```

#### Input Parameters
| Parameter | Location     | Type    | Description                                                  |
|-----------|--------------|---------|--------------------------------------------------------------|
| venueId   | Path         | string  | Venue identifier                                             |
| tableId | Path | string  | Table identifier. |


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
| deposit | [deposit](./Deposit.md) | Deposit paid by guest for confirming the reservation | 
 

#### Example

Example request:
```bash
curl --location --request POST 'https://wr-api.wellet.dev/venues/acme/table/12/open' \
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
    "conciergeName": "Valeria Pérez",
    "deposit": [
        "amount": 1500,
        "currency": "MXN",
        "transactions": [
            {
                "id": 23,
                "amount": 1100.00,
                "currency": "MXN",
                "paidAtUtc": "2024-01-02T13:23:58"
            },
            {
                "id": 25,
                "amount": 400.00,
                "currency": "MXN",
                "paidAtUtc": "2024-01-02T13:28:58"
            }
        ]
    ]
}
```

#### Error Codes
The following HTTP Status Codes can be returned by this endpoint:

| Code      | Status    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| 404       | Not Found | The venue or table was not found. The error description returned is one of: `TABLE_NOT_FOUND`, `VENUE_NOT_FOUND`.|

Example error responses:

```json
{
    "code": 404,
    "errorDescription": "TABLE_NOT_FOUND"
}
```

```json
{
    "code": 404,
    "errorDescription": "VENUE_NOT_FOUND"
}
```

## Register a payment for a reservation by table id
Registers a new payment for a particular table. If this endpoint is called multiple times with different payment ids for the same table, multiple payments will be added to the reservation. Note: if two payments are registered with the same id for the same reservation, only the first one will be registered, as the api assumes that this is a duplicated record sent by accident.

```
PUT /venues/{venueId}/table/{tableId}/payment
```

#### Input Parameters
| Parameter | Location     | Type    | Description                                                  |
|-----------|--------------|---------|--------------------------------------------------------------|
| venueId   | Path         | string  | Venue identifier                                             |
| tableId | Path | string  | Table identifier. |
| paymentId      | Body | string  | Any string identifying the payment (optional). |
| amount      | Body | number  | Amount paid. |
| currency      | Body | string  | Currency of payment. Possible values: 'MXN' |
| tableNumber | Body | string | Table number or identifier, as it is displayed to the final user.|
| paxs | Body | number | Number of people in the group.|
| hasDiscount | Body | boolean | True if a discount was applied to this table. (Optional, false by default) |
| discountName | Body | string | Name of the discount applied (optional). |
| discountAmount | Body | number | The amount of the discount applied, if any. (Optional, default: 0)|
| checkNumber | Body | string | The check nunber of the table. (optional)|
| checkUrl | Body | string | Check url containing a PDF file or image of the check (optional)|
| events | Body | [events](./Events.md) | Date and user information about events detailed [here](./Events.md). (Optional)|
| reservationDate | Body | string | Reservation date. Format: "yyyy-mm-dd". Optional, only needed when reservation is closed and it is needed to patch payments not registered for some reason.|

#### Output Parameters
If the payment was successfully registered, the following parameters are returned:

| Parameter | Type    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| code      | string  | Reservation code                                       |
| date      | string  | Reservation date in "yyyy-MM-dd" format.               |
| time      | string  | Reservation time in "HH:mm" 24-hour notation format.   |
| paxs      | int     | Number of people in the group.                         |
| tableId      | string     | Table identifier                         |
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
curl --location --request PUT 'https://wr-api.wellet.dev/venues/acme/table/12/payment' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "paymentId": "ABC123",
    "amount": 15383.25,
    "currency": "MXN",
    "paxs": 8,
    "hasDiscount": true,
    "discountName": "Locales",
    "discountAmount": 1709.25,
    "checkNumber": "205",
    "checkUrl": "https://billing-xyz.com/checks/check205.pdf",
}'
```

Example response:
```json
{
    "code": "GFAL",
    "date": "2024-01-04",
    "time": "20:30",
    "paxs": 8,
    "tableId": 12,
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

