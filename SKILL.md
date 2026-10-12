---
name: nightz-booking
description: Book nights at Nightz properties on Base. Use when a user wants to browse Nightz properties (Casa Akaw in La Saladita, Tulum Penthouse), check availability, buy night tokens from a launch sale, burn tokens to book dates, check balances or bookings, or manage affiliate referrals.
---

## When to Use / When NOT to Use

**Use when:**
- User wants to browse Nightz properties (Casa Akaw, Tulum Penthouse)
- Checking availability for specific dates
- Buying nights from a property's launch sale
- Making a reservation (hold dates → burn tokens → confirm)
- Checking token balance, sale status or bookings
- Affiliate/referral questions

**Don't use when:**
- General real estate or other vacation rentals
- Price speculation or investment advice. Nights are for stays, not an investment product.

# Nightz Booking

1 token = 1 night at one specific property. Buy at the fixed launch price → hold dates → burn → booking confirmed. Tokens are transferable; Casa Akaw also has a small Uniswap V3 pool.

## Properties (Base, chainId 8453)

| | Casa Akaw | Tulum Penthouse |
|---|---|---|
| Location | La Saladita, Guerrero, Mexico | Selva Norte, Tulum, Mexico |
| `propertyCode` (booking API) | `TULUM03` (legacy code) | `TULUM_PENTHOUSE` |
| Token | NZAKAW `0x1D0b7B031f12F7171aa6b1D92684Fa3d899b3846` | NZTULUM `0x2Aaa8a37F9FD792EFcb243579850d9A88DF3ca1b` |
| Launch sale (V2) | `0x40f26130b66C643EaCb1870fDE5c03185c75459d` | `0x51444B7ca8303D4F12D1531b3fe2F834774799c1` |
| Launch price | $80 USDC / night | $130 USDC / night |
| Uniswap V3 | NZAKAW/USDC pool `0xa652F17734B88B79aD2a36D53bb0E427992f88Eb` (1%) | No pool yet |

**USDC (Base):** `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` (6 decimals)
**Token decimals:** 2 → 1 night = 100 raw units
**Affiliate:** 8% of USDC paid in `buy()` (800 bps), per sale contract

Always read live values (`costFor`, `totalSold`, `paused`) instead of trusting this table.

## Launch Sale ABI (key functions)

```json
[
  "function buy(uint256 amount, address referrer) external",
  "function claimReferral() external",
  "function currentPrice() view returns (uint256)",
  "function totalSold() view returns (uint256)",
  "function maxSupply() view returns (uint256)",
  "function costFor(uint256 amount) view returns (uint256)",
  "function paused() view returns (bool)",
  "function referralBps() view returns (uint256)",
  "function pendingReferral(address) view returns (uint256)",
  "function referrerOf(address) view returns (address)"
]
```

## Token ABI (key functions)

```json
[
  "function balanceOf(address) view returns (uint256)",
  "function decimals() view returns (uint8)",
  "function burn(uint256 amount) external",
  "function transfer(address to, uint256 amount) returns (bool)"
]
```

## Functions

### 1. Check Sale Status
Read `totalSold()`, `maxSupply()`, `currentPrice()`, `paused()` on the property's sale contract.
- Remaining nights = (maxSupply − totalSold) / 100
- Price = currentPrice() / 1e6 USD

### 2. Buy Nights
1. `amount = nights × 100`; get cost with `sale.costFor(amount)`
2. `usdc.approve(sale, cost)` (exact amount only)
3. `sale.buy(amount, referrer)`, `referrer = address(0)` if none
4. The first referrer is locked per buyer, per sale. Self-referral earns nothing.

### 3. Check Balance
`token.balanceOf(wallet) / 100` = nights held for that property.

### 4. Book Dates (hold → burn → confirm)

**Never burn before a hold exists.** Burns are irreversible and there is no on-chain refund.

1. **Availability**: `GET /availability?property=TULUM03&from=YYYY-MM-DD&to=YYYY-MM-DD` → `{ available: true|false }`
2. **Hold**: wallet-sign and `POST /reservation-holds` with exactly:
   ```json
   { "property": "TULUM03", "wallet": "0x…", "from": "2027-04-10", "to": "2027-04-13", "nights": 3, "name": "Guest Name", "email": "guest@example.com" }
   ```
   → `{ "id": "<holdId>", "expiresAt": "…" }`. The hold lasts 30 minutes and checks the wallet's token balance.
3. **Burn**: `token.burn(nights × 100)` from the same wallet while the hold is active. Wait for 2 confirmations.
4. **Confirm**: wallet-sign and `POST /reservations` with the same seven fields plus `holdId` and `txHash`. The server verifies the burn on-chain (chain 8453, same wallet, exact amount, correct token, inside the hold window).
5. If confirmation fails, **retry with the same txHash**. Never burn again for the same stay.

Rules: minimum 2 nights, check-in from tomorrow, checkout date excluded, check-in 15:00 / check-out 11:00. ID and a security deposit may be requested on arrival.

### 5. Wallet Authorization (EIP-191)

Booking writes and private reads need a wallet signature (externally owned wallets only; EIP-1271 not supported). Headers:
- `X-Wallet-Issued-At`: 13-digit ms timestamp (valid 5 minutes)
- `X-Wallet-Signature`: `signMessage` result

Exact message (lowercase wallet; payload = JSON with keys sorted alphabetically):

```
Nightz wallet authorization
Domain: nightz.co
Chain: 8453
Action: reservations.hold
Wallet: 0xabc…
Issued at: 1767225600000
Payload: {"email":"…","from":"…","name":"…","nights":3,"property":"TULUM03","to":"…","wallet":"0x…"}
```

Actions: `reservations.hold`, `reservations.create` (payload = the full request body), `reservations.read` (payload = `{"wallet":"0x…lowercase"}`).

### 6. Affiliate / Referral
- Share `https://nightz.co?ref=0xAFFILIATE` or a registered `/r/CODE` link
- Commissions accrue per sale contract; check `pendingReferral(address)` on each and call `claimReferral()` where > 0
- Commissions come only from launch-sale purchases, not from bookings

## API

- **Base URL:** `https://nightz-api.onrender.com`
- Booking endpoints (`/availability`, `/reservation-holds`, `/reservations`) use wallet signatures, no API key.
- Agent endpoints under `/agent` need header `X-Agent-Key: <key>` (request access via nightz.co).

| Method | Path | Purpose |
|--------|------|---------|
| GET | /agent/info | How the system works |
| GET | /agent/properties | List properties (with `propertyCode`, token, sale) |
| GET | /agent/properties/:id | `akaw` or `tulum` |
| GET | /agent/properties/:id/availability?from=&to= | Available dates in a range |
| GET | /agent/bookings/:wallet | Reservations (also needs wallet signature) |
| GET | /agent/affiliate/:address | Pending commission across both sales |
| GET | /availability?property=&from=&to= | Is a date range free |
| POST | /reservation-holds | Hold dates for 30 minutes |
| POST | /reservations | Confirm after burn |

`POST /agent/book` was retired (410). Use the hold → burn → confirm flow.

## For More Details
See [references/api-response-examples.md](references/api-response-examples.md) for response schemas.
