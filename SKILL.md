---
name: nightz-booking
description: Book tokenized property nights via the Nightz platform. Use when a user wants to browse vacation properties, check availability, buy NZAKAW tokens from launchpad, burn tokens to book dates, swap on secondary market, check balances, or manage affiliate referrals. Handles the full booking flow for Casa Akaw (La Saladita, Mexico).
---

## When to Use / When NOT to Use

**Use when:**
- User wants to browse Nightz properties (Casa Akaw, La Saladita)
- Checking availability for specific dates
- Buying NZAKAW from the launchpad (primary sale)
- Making a reservation (burn tokens → book)
- Checking token balance, launchpad status, or bookings
- Swapping NZAKAW on secondary market (Uniswap V3)
- Affiliate/referral questions

**Don't use when:**
- General real estate or other vacation rentals
- Crypto trading/market analysis → **crypto-monitoring**
- Content creation → **tododecripto-content**

# Nightz Booking

1 NZAKAW token = 1 night. Buy from launchpad or secondary market → burn on-chain → book dates.

## Contracts (Base, chainId 8453)

| Contract | Address |
|----------|---------|
| **NightzLaunchpadV2** | `0x40f26130b66C643EaCb1870fDE5c03185c75459d` |
| **NZAKAW Token** | `0x1D0b7B031f12F7171aa6b1D92684Fa3d899b3846` |
| **USDC (Base)** | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` |

**Token decimals:** 2 (1 night = 100 smallest units)
**Price:** $80 USDC per night (both early & regular tier)
**Supply:** 1000 nights total
**Affiliate:** 8% commission (800 bps)

## Launchpad V2 ABI (key functions)

```json
[
  "function buy(uint256 amount, address referrer) external",
  "function claimReferral() external",
  "function currentPrice() view returns (uint256)",
  "function isEarlyBird() view returns (bool)",
  "function totalSold() view returns (uint256)",
  "function maxSupply() view returns (uint256)",
  "function earlyRemaining() view returns (uint256)",
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
  "function approve(address spender, uint256 amount) returns (bool)",
  "function transfer(address to, uint256 amount) returns (bool)"
]
```

## Functions

### 1. Check Launchpad Status
Read `totalSold()`, `maxSupply()`, `currentPrice()`, `isEarlyBird()`, `paused()` on the launchpad contract.
- Remaining = maxSupply - totalSold
- Price = currentPrice() (returns USDC with 6 decimals, divide by 1e6 for USD)

### 2. Buy Nights from Launchpad
1. User approves USDC spending: `usdc.approve(launchpad, cost)`
2. Call `launchpad.buy(amount, referrer)` where amount = nights × 100 (2 decimals)
3. Use `address(0)` for referrer if none, or pass affiliate address
4. Cost can be checked with `costFor(amount)`
5. First referrer is locked permanently per buyer

### 3. Check Token Balance
Call `nzakaw.balanceOf(wallet)` → divide by 100 for nights count.

### 4. Reserve / Book Dates (Burn Tokens)
1. **Check availability** → `GET /agent/properties/akaw/availability?from=YYYY-MM-DD&to=YYYY-MM-DD`
2. **Burn tokens** → `nzakaw.burn(nights * 100)`
3. **Book** → `POST /agent/book` with burn txHash

### 5. Swap on Secondary Market (Uniswap V3)
For buying/selling after launchpad sells out. Use Uniswap V3 on Base.
- Pool: NZAKAW/WETH or check DexScreener for active pools
- Frontend: nightz.io links to Uniswap swap page

### 6. Affiliate / Referral System
- 8% of USDC paid goes to referrer as commission
- Referrer is locked on buyer's first purchase
- Affiliates claim with `claimReferral()`
- Check pending: `pendingReferral(address)`
- Share buy link with `?ref=0xAFFILIATE_ADDRESS`

## API (Booking Backend)

- **Base URL:** `https://nightz-api.onrender.com/agent`
- **Auth:** Header `X-Agent-Key: <key>`

| Method | Path | Purpose |
|--------|------|---------|
| GET | /agent/properties | List properties |
| GET | /agent/properties/:id | Property details |
| GET | /agent/properties/:id/availability?from=&to= | Available dates |
| POST | /agent/book | Create reservation after burn |
| GET | /agent/bookings/:wallet | View reservations |

### POST /agent/book Body
```json
{
  "propertyId": "akaw",
  "wallet": "0x...",
  "from": "2026-03-15",
  "to": "2026-03-18",
  "nights": 3,
  "guestName": "John Doe",
  "guestEmail": "john@example.com",
  "txHash": "0x..."
}
```

## Key Details

- **Check-in:** 15:00 / **Check-out:** 11:00
- **Min nights:** 2
- **Cancellation:** Irreversible (burn cannot be undone)
- **Owner controls:** pause/unpause, withdraw USDC, recover unsold tokens

## For More Details
See [references/api-response-examples.md](references/api-response-examples.md) for full response schemas.
