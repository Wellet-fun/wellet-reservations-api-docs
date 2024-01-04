# Wellet Reservations API Reference
This repository contains documentation for using the [Wellet](https://wellet.us/) Reservations API for obtaining information and updating reservations made through the Wellet platform.

This api is organized around [REST](https://en.wikipedia.org/wiki/REST), accepts [JSON](https://www.json.org/json-en.html) request bodies, returns [JSON-encoded](https://www.json.org/json-en.html) responses, and uses standard HTTP response codes and verbs.

## Base URL

The Reservations API is served over HTTPs. All URLs referenced in the documentation have the following base:
```
https://wr-api.wellet.cloud
```

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
curl --location --request GET 'https://wr-api.wellet.cloud/venues/' \
--header 'x-api-key: YOUR_API_KEY'
```

Example response:
```json
[
    {
        "id": "chambao-cancun",
        "name": "Chambao Cancún",
    },
    {
        "id": "chambao-tulum",
        "name": "Chambao Tulum",
    },
    {
        "id": "parole-cancun",
        "name": "Parole Cancún",
    },
    {
        "id": "parole-tulum",
        "name": "Parole Tulum",
    }
]
```
## Get Venue Reservations
Returns the list of reservations confirmed for a particular venue and a particular date.

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
| conciergeName | string | Name of the concierge that generated the reservation |

#### Example

Example request:
```bash
curl --location --request GET 'https://wr-api.wellet.cloud/venues/chambao-cancun/reservations?date=2024-01-04' \
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
        "conciergeName": "Diego Sánchez"
    },
    {
        "code": "GFAL",
        "date": "2024-01-04",
        "time": "20:30",
        "paxs": 8,
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
| conciergeName | string | Name of the concierge that generated the reservation |

#### Example

Example request:
```bash
curl --location --request GET 'https://wr-api.wellet.cloud/venues/chambao-cancun/reservations/GFAL' \
--header 'x-api-key: YOUR_API_KEY'
```

Example response:
```json
{
    "code": "GFAL",
    "date": "2024-01-04",
    "time": "20:30",
    "paxs": 8,
    "conciergeName": "Valeria Pérez"
}
```
#### Error Codes
The following HTTP Status Codes can be returned by this endpoint:

| Code      | Status    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| 404       | Not Found | The reservation was not found or it is  not confirmed (for example: it has been cancelled).|

## Register a payment for a reservation
Registers a new payment for a particular reservation. If this endpoint is called multiple times for the same reservation code, the payments will be added to the reservation.

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

#### Output Parameters
If the payment was successfully registered, the following parameters are returned:

| Parameter | Type    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| code      | string  | Reservation code                                       |
| date      | string  | Reservation date in "yyyy-mm-dd" format.               |
| time      | string  | Reservation time in "hh:mm" 24-hour notation format.   |
| paxs      | int     | Number of people in the group.                         |
| totalPaidAmount | number | Total amount paid for this order (sum of all payments received) |
| currency      | string  | Currency of payments. |

#### Error Codes
The following HTTP Status Codes can be returned by this endpoint:

| Code      | Status    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| 200       | OK      | The payment was successfully registered. |
| 404       | Not Found | The reservation was not found or it is  not confirmed (for example: it has been cancelled).|

#### Example

Example request:
```bash
curl --location --request PUT 'https://wr-api.wellet.cloud/venues/chambao-cancun/reservations/GFAL/payment' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "paymentId": "ABC123",
    "amount": 15383.25,
    "currency": "MXN"
}'
```

Example response:
```json
{
    "code": "GFAL",
    "date": "2024-01-04",
    "time": "20:30",
    "paxs": 8,
    "totalPaidAmount": 15383.25,
    "currency": "MXN"
}
```
