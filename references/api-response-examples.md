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
      "images": ["https://nightz-frontend.vercel.app/akaw-1.jpg"],
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
        "referenceNightPriceUsd": 220,
        "tokensPerNight": 1,
        "currency": "NZAKAW"
      },
      "rules": {
        "minNights": 1,
        "maxNights": 30,
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
      "status": "active"
    }
  ],
  "count": 1
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
    "step1": "Acquire NZAKAW tokens (buy on Uniswap or OTC)",
    "step2": "POST /agent/book with dates, wallet, guest info, and burn txHash",
    "step3": "Tokens are burned on-chain, reservation is confirmed",
    "tokenAddress": "0x1D0b7B031f12F7171aa6b1D92684Fa3d899b3846",
    "tokensPerNight": 1,
    "buyUrl": "https://app.uniswap.org/swap?chain=base..."
  }
}
```

## POST /agent/book — Success (201)

```json
{
  "success": true,
  "reservation": {
    "id": "m1abc123-4def5678",
    "property": "akaw",
    "propertyName": "Casa Akaw",
    "wallet": "0x...",
    "from": "2026-03-15",
    "to": "2026-03-18",
    "nights": 3,
    "status": "Confirmed",
    "guestName": "John Doe",
    "guestEmail": "john@example.com",
    "txHash": "0x..."
  },
  "message": "Reservation confirmed for 3 night(s) at Casa Akaw.",
  "checkIn": "15:00",
  "checkOut": "11:00"
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
