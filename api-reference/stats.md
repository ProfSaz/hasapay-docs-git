# Stats API

Aggregate counters for the org dashboard — wallet balance, volume, transaction counts, gas spend.

**Authentication:** Dual-auth (JWT or HMAC)
**Base path:** `/api/v1/stats`

> All three stats endpoints accept `network_type=testnet|mainnet` to scope results to one environment. Defaults to `testnet`.

---

## Get organization stats

```
GET /api/v1/stats?network_type=testnet
```

Top-line counters for the dashboard "home" page.

### Query params

| Param | Default | Description |
|---|---|---|
| `network_type` | `testnet` | `testnet` or `mainnet` |

### Response

```json
{
  "success": true,
  "data": {
    "current_balance_usd": 12450.75,
    "total_balance_usd": 145200.50,
    "deposit_volume_usd": 234500.00,
    "withdrawal_volume_usd": 89000.00,
    "deposit_count": 412,
    "withdrawal_count": 87,
    "wallet_count": 5,
    "address_count": 1240,
    "asset_count": 8,
    "team_member_count": 4,
    "gas_fees_usd": 312.40,
    "balance_change_30d": 12.3,
    "volume_change_30d": -4.1,
    "last_deposit_at": "2026-06-09T11:45:00Z",
    "last_withdrawal_at": "2026-06-09T09:30:00Z",
    "network_type": "testnet"
  }
}
```

### Field reference

| Field | Meaning |
|---|---|
| `current_balance_usd` | **Real portfolio value** — `SUM(account_balances) × current USD price`. The number to show users. |
| `total_balance_usd` | Legacy lifetime counter (`deposits − withdrawals − gas`). Not actual holdings — kept for back-compat. |
| `deposit_volume_usd` / `withdrawal_volume_usd` | Lifetime USD-denominated flow on the chosen `network_type` |
| `deposit_count` / `withdrawal_count` | Lifetime tx counts |
| `wallet_count` / `address_count` | Currently active master wallets + child addresses |
| `asset_count` | Assets currently enabled on the org |
| `team_member_count` | Active members (excludes deactivated) |
| `gas_fees_usd` | Lifetime gas spend on this network type |
| `balance_change_30d` / `volume_change_30d` | Percentage delta vs. the prior 30-day window |
| `last_deposit_at` / `last_withdrawal_at` | Most recent transaction timestamps |
| `network_type` | Echo of the query param so the client can confirm what was returned |

---

## Get volume history

```
GET /api/v1/stats/volume?network_type=testnet&days=30
```

Daily breakdown for a line/bar chart.

### Query params

| Param | Default | Description |
|---|---|---|
| `network_type` | `testnet` | `testnet` or `mainnet` |
| `days` | `30` | Window in days (capped at 365) |

### Response

```json
{
  "success": true,
  "data": {
    "network_type": "testnet",
    "days": 30,
    "volume": [
      {
        "date": "2026-05-11",
        "deposit_volume_usd": 1200.00,
        "withdrawal_volume_usd": 450.00,
        "deposit_count": 5,
        "withdrawal_count": 2
      },
      {
        "date": "2026-05-12",
        "deposit_volume_usd": 800.00,
        "withdrawal_volume_usd": 0.00,
        "deposit_count": 3,
        "withdrawal_count": 0
      }
    ]
  }
}
```

`volume[]` is ordered ascending by date and is dense — days with zero activity are returned with all-zero values so the chart x-axis stays consistent.

---

## Get chain breakdown

```
GET /api/v1/stats/chains?network_type=testnet
```

Per-chain split of volume + counts + gas spend.

### Query params

| Param | Default | Description |
|---|---|---|
| `network_type` | `testnet` | `testnet` or `mainnet` |

### Response

```json
{
  "success": true,
  "data": {
    "network_type": "testnet",
    "chains": [
      {
        "chain": "ethereum",
        "deposit_volume_usd": 145000.00,
        "withdrawal_volume_usd": 58000.00,
        "deposit_count": 210,
        "withdrawal_count": 41,
        "gas_fees_usd": 198.30
      },
      {
        "chain": "polygon",
        "deposit_volume_usd": 89500.00,
        "withdrawal_volume_usd": 31000.00,
        "deposit_count": 202,
        "withdrawal_count": 46,
        "gas_fees_usd": 12.80
      }
    ]
  }
}
```

---

## Notes

- All USD numbers use **current** prices from the price service; historical figures are not back-priced to their tx-time exchange rate.
- Sweeps are internal movements and **do not** contribute to `deposit_volume_usd` or `withdrawal_volume_usd`.
- `gas_fees_usd` covers gas paid by HasaPay master wallets (sends + sweeps); deposit-side gas is paid by the sender and not counted here.

---

## Errors

| Code | Cause |
|---|---|
| `Organization not found in context` | No auth — provide JWT or HMAC headers |
| `Failed to get stats: ...` | Underlying query/price-service failure; treat as retryable |
