# Fees API

Configure fees, preview them before sending, and pull historical fee analytics.

**Authentication:** Dual-auth (JWT or HMAC)
**Base path:** `/api/v1/fees`

---

## Fee resolution chain

When HasaPay calculates a fee, it walks a five-tier lookup and uses the first match:

1. **Address override** — `(child_address)` — most specific
2. **Org × chain × network × token** — exact match in `fee_configurations`
3. **Org × chain × network** — wildcard across tokens
4. **Org × chain** — wildcard across networks
5. **Platform defaults** — `chain_defaults.go` per-chain table

Per-chain platform defaults (network: `mainnet` unless noted):

| Chain | Withdrawal fee | Deposit fee | Min deposit |
|---|---|---|---|
| Ethereum | 1% | 1% | $10 |
| Polygon | 0.5% | 0.5% | $1 |
| BSC | 0.5% | 0.5% | $2 |
| Base | 0.5% | 0.5% | $1 |
| Tron | flat $5 | flat $5 | $20 |
| All testnets | 0% | 0% | $0 |

> **Rate convention.** Percentage rates are stored as decimals — `0.02 = 2%`. Multiply by 100 for display. The one exception is `effective_fee_rate` in some response payloads, which is pre-multiplied.

> **`min_deposit_usd` is "accept-and-defer", not reject.** A deposit below the threshold is still credited to the ledger; the sweep is just deferred until accumulated balance crosses the threshold.

---

## Get fee config

```
GET /api/v1/fees/config
```

Returns the org's resolved fee configuration — platform defaults, org overrides, gasless flags, rev-share settings. Use it to render a "current fees" panel.

### Example response

```json
{
  "data": {
    "deposit_fee": { "enabled": true, "type": "percentage", "rate": 0.005 },
    "withdrawal_fee": { "enabled": true, "type": "percentage", "rate": 0.01 },
    "org_fee": {
      "enabled": false,
      "deposit_fee_type": "percentage",
      "deposit_fee_rate": 0,
      "withdrawal_fee_type": "percentage",
      "withdrawal_fee_rate": 0
    },
    "rev_share": { "enabled": false, "percentage": 0 },
    "gasless": { "enabled": true }
  }
}
```

---

## Address fee overrides

Per-(child address) overrides. Useful when one customer's payment terms differ from your org default.

### List overrides

```
GET /api/v1/fees/addresses
```

```json
{
  "data": {
    "overrides": [ /* AddressFeeOverride[] */ ],
    "total": 3
  }
}
```

### Get one override

```
GET /api/v1/fees/addresses/:addressId
```

### Set / upsert an override

```
PUT /api/v1/fees/addresses/:addressId
```

#### Request body — all fields optional, send only what you want to change

| Field | Type | Description |
|---|---|---|
| `fees_enabled` | bool | Master switch — `false` disables all fees on this address |
| `deposit_fee_override` | bool | Set true to use the address-level deposit fee fields instead of org defaults |
| `deposit_fee_type` | string | `percentage` or `flat` |
| `deposit_fee_rate` | float | Decimal rate (`0.02 = 2%`) or flat amount |
| `deposit_fee_min` | string | Min absolute fee (token units) |
| `deposit_fee_max` | string | Max absolute fee cap |
| `withdrawal_fee_override` | bool | Same idea for withdrawals |
| `withdrawal_fee_type` | string | `percentage` or `flat` |
| `withdrawal_fee_rate` | float | Rate or flat amount |
| `withdrawal_fee_min` / `max` | string | Same as deposit |
| `gasless_enabled` | bool | Address-level gasless toggle |
| `notes` | string | Free-text label |

### Delete an override (revert to org default)

```
DELETE /api/v1/fees/addresses/:addressId
```

---

## Withdrawal fee estimate

```
GET /api/v1/fees/estimate?token=USDC&chain=ethereum&network=sepolia&amount=100
```

Server-side fee quote for a withdrawal. Walks the full resolution chain and returns gas + protocol + total broken out.

### Query params

| Param | Required | Description |
|---|---|---|
| `token` | Yes | Token symbol (e.g. `USDC`) |
| `chain` | Yes | Chain |
| `network` | Yes | Network |
| `amount` | Yes | Human-readable amount (e.g. `100` for 100 USDC) |
| `address_id` | No | Child address UUID — applies any per-address override |

### Response

```json
{
  "data": {
    "token": "USDC",
    "chain": "ethereum",
    "network": "sepolia",
    "send_amount": {
      "amount": "100",
      "amount_raw": "100000000",
      "token": "USDC"
    },
    "protocol_fee": {
      "amount": "1.00",
      "amount_raw": "1000000",
      "token": "USDC",
      "breakdown": {
        "platform_fee": { "amount": "1.00", "rate": 0.01 },
        "org_fee": { "amount": "0", "rate": 0 }
      }
    },
    "gas_fee": {
      "native_symbol": "ETH",
      "native_amount": "0.0008",
      "usd": 2.10,
      "token_equivalent": "2.10",
      "token": "USDC",
      "buffer_pct": 30
    },
    "total_fee": {
      "amount": "3.10",
      "amount_usd": 3.10,
      "token": "USDC"
    },
    "total_deducted": {
      "amount": "103.10",
      "amount_raw": "103100000",
      "token": "USDC"
    },
    "quote_valid_secs": 60
  }
}
```

- `send_amount` = what the recipient receives
- `total_deducted` = what comes out of your balance
- `gas_fee.buffer_pct` = the safety buffer added to live gas (30% by default)
- `quote_valid_secs` = how long this estimate is reliably accurate before gas drifts

---

## Deposit fee estimate

```
GET /api/v1/fees/deposit/estimate?token=USDC&chain=ethereum&network=sepolia&amount=100
```

Mirror of the withdrawal estimate, but for incoming deposits. Used to show "you will receive X" before a customer sends.

### Query params

Same as the withdrawal estimate.

### Response (key extras)

```json
{
  "data": {
    "token": "USDC",
    "chain": "ethereum",
    "network": "sepolia",
    "amount": {
      "amount": "100",
      "amount_raw": "100000000",
      "amount_usd": 100.00,
      "token": "USDC"
    },
    "protocol_fee": {
      "amount": "1.00",
      "amount_raw": "1000000",
      "amount_usd": 1.00,
      "token": "USDC",
      "breakdown": {
        "platform_fee": { "amount": "1.00", "amount_raw": "1000000", "amount_usd": 1.00, "rate": 0.01 },
        "org_fee": { "amount": "0", "amount_raw": "0", "amount_usd": 0, "rate": 0 }
      }
    },
    "rev_share_earned": { "amount": "0", "amount_raw": "0", "token": "USDC" },
    "net_received": { "amount": "99.00", "amount_raw": "99000000", "token": "USDC" },
    "fee_source": "platform_default",
    "min_deposit_usd": 10,
    "below_min_deposit": false,
    "quote_valid_secs": 60
  }
}
```

| Field | Meaning |
|---|---|
| `net_received` | Amount credited to the customer's wallet after fees |
| `fee_source` | Which tier resolved (`address_override`, `org_chain_token`, `chain_default`, `platform_default`, etc.) |
| `min_deposit_usd` | The applicable minimum deposit threshold |
| `below_min_deposit` | Hint flag — if `true`, the deposit will be **accepted but the sweep deferred** until accumulated balance crosses the threshold |

---

## Fee analytics

### Summary

```
GET /api/v1/fees/summary?period=current_month&chain=ethereum
```

Aggregate fees over a period, broken down by chain, token, and sub-period.

#### Query params

| Param | Default | Description |
|---|---|---|
| `period` | `current_month` | `current_month`, `last_month`, `current_quarter`, `last_quarter`, `last_30d`, `last_90d`, `ytd` |
| `from_date` / `to_date` | — | Override `period` with explicit ISO dates |
| `chain` | — | Filter by chain |
| `token` | — | Filter by token symbol |
| `network_type` | `testnet` | `testnet` or `mainnet` |

#### Response

```json
{
  "data": {
    "period": { "label": "June 2026", "from": "2026-06-01", "to": "2026-06-30" },
    "totals": {
      "deposit_count": 412,
      "deposit_volume_usd": 234500.00,
      "platform_fees_paid_usd": 2345.00,
      "org_fees_earned_usd": 0,
      "rev_share_earned_usd": 0,
      "net_cost_usd": 2345.00
    },
    "by_chain": [
      { "chain": "ethereum", "deposit_volume_usd": 145000.00, "platform_fees_paid_usd": 1450.00 }
    ],
    "by_token": [
      { "token": "USDC", "deposit_volume_usd": 200000.00, "platform_fees_paid_usd": 2000.00 }
    ],
    "by_period": [
      { "label": "2026-06-01", "deposit_volume_usd": 7500.00, "platform_fees_paid_usd": 75.00 }
    ]
  }
}
```

### History

```
GET /api/v1/fees/history?page=1&limit=50
```

Per-deposit rows with attached fees + running period totals. Use this for an "activity log" view rather than aggregate summaries.

### Sources (per-wallet ranking)

```
GET /api/v1/fees/sources?sort_by=volume&period=current_month
```

Ranks child addresses + master wallets by activity — useful for spotting "this one customer is generating 90% of my fees" patterns.

#### Query params

| Param | Default | Description |
|---|---|---|
| `sort_by` | `volume` | `volume`, `fees_paid`, `tx_count`, `last_active` |
| `period` | `current_month` | Same period values as summary |
| `chain` / `token` / `network_type` | — | Standard filters |
| `limit` / `offset` | 50 / 0 | Pagination |

### Single source detail

```
GET /api/v1/fees/sources/:source_id
```

`:source_id` is either a child address ID or a master wallet ID — the handler resolves which.

Response includes the same per-chain/per-token/per-period breakdowns as the summary, scoped to that one source, plus a recent-activity log.

---

## Fee fields reference

| Field name | Format | Notes |
|---|---|---|
| `*_rate` | float (decimal) | `0.005 = 0.5%`. Multiply by 100 to display |
| `effective_fee_rate` | float (percentage) | **Pre-multiplied** — already in % form. Don't multiply again |
| `*_fee_type` | string | `percentage` or `flat` |
| `*_min`, `*_max` | string | Absolute amounts in token units |
| `*_fees_paid_usd` | float | Aggregated in USD using current prices |
| `net_cost_usd` | float | `fees_paid_usd − rev_share_earned_usd` |

---

## Errors

| Code | Cause |
|---|---|
| `validation_error` | Bad query params, invalid fee type string, token not supported on (chain, network) |
| `not_found` | Address override doesn't exist (only for explicit GET / DELETE — PUT auto-creates) |
| `forbidden` | Address doesn't belong to your organization |
| `internal_error` | Underlying service or price-feed failure — retryable |
