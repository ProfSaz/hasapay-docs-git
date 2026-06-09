# Wallets API

Create and manage HD master wallets across multiple blockchains. Master wallets are the parent of all child addresses on a chain/network.

**Authentication:**
- **HMAC-only** for `POST /wallets`
- **Dual-auth (JWT or HMAC)** for all reads and balance lookups

**Base path:** `/api/v1/wallets`

---

## Create master wallet

```
POST /api/v1/wallets
```
**Auth:** HMAC-only.

### Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `chain` | string | Yes | `ethereum`, `polygon`, `tron`, `base`, `bsc`, `solana`, `bitcoin`, `aptos` |
| `network` | string | Yes | `mainnet`, `sepolia`, `amoy`, `shasta`, etc. — see [supported networks](#supported-chains-and-networks) |
| `label` | string | No | Friendly name |

### Example

```bash
curl -X POST https://apitest.hasapay.com/api/v1/wallets \
  -H "X-API-Key: $API_KEY" \
  -H "X-Signature: $SIG" \
  -H "X-Timestamp: $TS" \
  -H "X-Request-ID: $RID" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": "ethereum",
    "network": "sepolia",
    "label": "Main ETH Wallet"
  }'
```

### Response

```json
{
  "data": {
    "id": "uuid",
    "organization_id": "uuid",
    "chain": "ethereum",
    "network": "sepolia",
    "address": "0xd502b72b8D969D60D1094174b1457C73671eb9d8",
    "label": "Main ETH Wallet",
    "is_active": true,
    "child_count": 0,
    "created_at": "2026-06-09T10:00:00Z",
    "updated_at": "2026-06-09T10:00:00Z"
  }
}
```

> Only one master wallet per `(organization, chain, network)`. Retrying create returns the existing wallet, not an error.

---

## List master wallets

```
GET /api/v1/wallets
```
**Auth:** Dual-auth.

### Query params

| Param | Default | Description |
|---|---|---|
| `include_balances` | `false` | Inline current balances per wallet |

### Response

```json
{
  "data": [
    {
      "id": "uuid",
      "chain": "ethereum",
      "network": "sepolia",
      "address": "0xd502b72b...",
      "label": "Main ETH Wallet",
      "is_active": true,
      "child_count": 5,
      "balances": [
        {
          "asset_id": "uuid",
          "token_symbol": "ETH",
          "balance": "0.5",
          "balance_raw": "500000000000000000",
          "is_stale": false,
          "last_synced": "2026-06-09T10:30:00Z"
        }
      ],
      "created_at": "2026-06-09T10:00:00Z"
    }
  ],
  "count": 3
}
```

---

## Get master wallet

```
GET /api/v1/wallets/:walletId
```
**Auth:** Dual-auth.

### Query params

| Param | Default | Description |
|---|---|---|
| `include_balances` | `false` | Inline balances |

Returns a single wallet in the same shape as the list rows.

---

## Get single-token balance

```
GET /api/v1/wallets/:walletId/balance
```
**Auth:** Dual-auth.

### Query params

| Param | Required | Description |
|---|---|---|
| `chain` | Yes | Chain of the asset |
| `network` | Yes | Network of the asset |
| `token` | Yes | Token symbol (e.g. `USDC`) |
| `token_address` | No | Contract address for ambiguous symbols (e.g. multiple USDCs on a chain) |

### Response

```json
{
  "data": {
    "chain": "ethereum",
    "network": "sepolia",
    "token_symbol": "USDC",
    "balance": "1000.00",
    "balance_raw": "1000000000",
    "pending_in": "0",
    "pending_in_raw": "0",
    "pending_out": "0",
    "pending_out_raw": "0"
  }
}
```

`pending_in` is the sum of unconfirmed deposits; `pending_out` is the sum of unconfirmed withdrawals/sweeps. Both let you display a "pending balance" without waiting for confirmations.

---

## Get all balances for a wallet

```
GET /api/v1/wallets/:walletId/balances
```
**Auth:** Dual-auth.

Returns every token balance the wallet holds on its chain/network.

```json
{
  "data": [
    {
      "asset_id": "uuid",
      "chain": "ethereum",
      "network": "sepolia",
      "token_symbol": "ETH",
      "token_decimals": 18,
      "balance": "0.5",
      "balance_raw": "500000000000000000",
      "is_stale": false,
      "last_synced": "2026-06-09T10:30:00Z"
    },
    {
      "asset_id": "uuid",
      "chain": "ethereum",
      "network": "sepolia",
      "token_symbol": "USDC",
      "token_decimals": 6,
      "balance": "1000.00",
      "balance_raw": "1000000000",
      "is_stale": false,
      "last_synced": "2026-06-09T10:30:00Z"
    }
  ]
}
```

---

## Supported chains and networks

| Chain | Mainnet | Testnet |
|---|---|---|
| `ethereum` | `mainnet` | `sepolia` |
| `polygon` | `mainnet` | `amoy` |
| `bsc` | `mainnet` | `testnet` |
| `base` | `mainnet` | `sepolia` |
| `tron` | `mainnet` | `shasta` |

Solana, Bitcoin, and Aptos are scaffolded but not generally available.

---

## Errors

| Code | Cause |
|---|---|
| `WALLET_NOT_FOUND` | No wallet with that ID on this org |
| `WALLET_ALREADY_EXISTS` | Returned existing — see note under create |
| `INVALID_CHAIN` | Unsupported chain string |
| `INVALID_NETWORK` | Unsupported network for this chain |
