# Nightz API Response Examples

## GET /agent/properties

```json
{
  "properties": [
    {
      "id": "akaw",
      "name": "Casa Akaw",
      "location": "La Saladita, Guerrero, Mexico",
      "description": "Beachfront surf house in one of Mexico's best surf spots.",
      "coordinates": { "lat": 17.7547, "lng": -101.7283 },
      "googleMapsUrl": "https://maps.app.goo.gl/R1SNgjx736feb5me9",
      "images": ["https://nightz.co/akaw-1.jpg"],
      "token": {
        "symbol": "NZAKAW",
        "address": "0x1D0b7B031f12F7171aa6b1D92684Fa3d899b3846",
        "chain": "base",
        "chainId": 8453,
        "decimals": 2,
        "totalSupply": 5000,
        "mechanism": "burn-to-book"
      },
      "pricing": {
        "referenceNightPriceUsd": 80,
        "tokensPerNight": 1,
        "currency": "NZAKAW"
      },
      "rules": {
        "minNights": 2,
        "maxNights": 365,
        "checkIn": "15:00",
        "checkOut": "11:00",
        "cancellation": "non-refundable (burn is irreversible)"
      },
      "uniswapPool": {
        "address": "0xa652F17734B88B79aD2a36D53bb0E427992f88Eb",
        "fee": 10000,
        "pair": "NZAKAW/USDC",
        "dexUrl": "https://app.uniswap.org/swap?chain=base..."
      },
      "sale": { "address": "0x40f26130b66C643EaCb1870fDE5c03185c75459d", "priceUsd": 80, "currency": "USDC" },
      "propertyCode": "TULUM03",
      "status": "active",
      "frontend": "https://nightz.co"
    },
    {
      "id": "tulum",
      "name": "Tulum Penthouse",
      "token": { "symbol": "NZTULUM", "address": "0x2Aaa8a37F9FD792EFcb243579850d9A88DF3ca1b", "chainId": 8453, "decimals": 2 },
      "pricing": { "referenceNightPriceUsd": 130, "tokensPerNight": 1, "currency": "NZTULUM" },
      "uniswapPool": null,
      "sale": { "address": "0x51444B7ca8303D4F12D1531b3fe2F834774799c1", "priceUsd": 130, "currency": "USDC" },
      "propertyCode": "TULUM_PENTHOUSE",
      "status": "active"
    }
  ],
  "count": 2
}
```

## GET /agent/properties/akaw/availability

```json
{
  "property": "akaw",
  "queryRange": { "from": "2026-03-01", "to": "2026-03-10" },
  "totalDaysInRange": 9,
  "availableDates": ["2026-03-01", "2026-03-02", "..."],
  "availableNights": 10,
  "bookedRanges": [
    { "from": "2026-03-05", "to": "2026-03-07", "nights": 2 }
  ],
  "bookingInstructions": {
    "step1": "Acquire NZAKAW tokens (launch sale 0x40f2… or Uniswap)",
    "step2": "Sign and POST /reservation-holds before burning (requires sufficient property tokens)",
    "step3": "Burn during the 30-minute hold, then sign and POST /reservations with holdId and txHash",
    "propertyCode": "TULUM03",
    "tokenAddress": "0x1D0b7B031f12F7171aa6b1D92684Fa3d899b3846",
    "tokensPerNight": 1,
    "buyUrl": "https://app.uniswap.org/swap?chain=base..."
  }
}
```

## GET /availability?property=TULUM03&from=2027-04-10&to=2027-04-13

```json
{ "property": "TULUM03", "from": "2027-04-10", "to": "2027-04-13", "available": true }
```

## POST /reservation-holds — Created (201)

```json
{ "id": "<holdId>", "expiresAt": "2027-04-01T18:30:00.000Z" }
```

## POST /reservations — Confirmed (201, or 200 on an idempotent retry)

Body: the hold fields plus `"holdId"` and `"txHash"` (burn of nights × 100). Response is the saved reservation with its `id`, `status` and `txHash`.

## POST /agent/book — Retired (410)

```json
{
  "error": "Booking protocol upgraded",
  "steps": ["Sign and POST /reservation-holds", "Burn during the hold window", "Sign and POST /reservations with holdId and txHash"]
}
```

## GET /agent/bookings/0x...

```json
{
  "wallet": "0x...",
  "totalReservations": 1,
  "reservations": [
    {
      "id": "m1abc123-4def5678",
      "property": "Casa Akaw",
      "wallet": "0x...",
      "from": "2026-03-15",
      "to": "2026-03-18",
      "nights": 3,
      "status": "Confirmed",
      "guestName": "John Doe",
      "guestEmail": "john@example.com",
      "txHash": "0x..."
    }
  ]
}
```
