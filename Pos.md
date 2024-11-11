# 1 - Get Bill (from mobile app to POS)

## Request

```javascript
GET {posURL}/bill?table=011
```

## Response

```javascript
{
  "table": "011",
  "checkId": "262623423",
  "items": [
    {
      "name": "Tequila Don Julio 70",
      "quantity": 2,
      "unitPrice": 300.0,
      "totalPrice": 600.0
    },
    {
      "name": "Ensalada Chambao",
      "quantity": 1,
      "unitPrice": 200.0,
      "totalPrice": 200.0
    },
    {
      "name": "Ojo de Bife 250gr.",
      "quantity": 2,
      "unitPrice": 900.0,
      "totalPrice": 1800.0
    },
  ],
  "subtotal": 2600.0,
  "taxes": 260.0,
  "totalAmount": 2860.0
}
```

# 2 - Pay (from mobile app to POS)

## Request
``` javascript
POST {posURL}/bill/pay

{
  "table": "011",
  "amount": 3380.0,
  "tip": 520.0,
  "payments": [
    {
        "type": "CARD",
        "card_type": "VISA",
        "last4": "1234",
        "expirationMonth": 5,
        "expirationYear": 26,
        "amount": 3120.00,
        "chargeId": "pi_1234"
    },
    {
        "type": "DISCOUNT",
        "amount": 260
    }
  ]
}
```

## Response

```
200 OK
```