# Addresses API

Child addresses (deposit / per-customer addresses) derived under a master wallet.

**Authentication:**
- **HMAC-only** for create, update, auto-sweep, send, estimate-gas
- **Dual-auth (JWT or HMAC)** for all reads

**Base paths:**
- Org-wide: `/api/v1/addresses`
- Wallet-scoped: `/api/v1/wallets/:walletId/addresses` (plus the singular create at `/wallets/:walletId/address`)

---

## Create address

> **Path note:** create is **singular** — `/address`, not `/addresses`.

```
POST /api/v1/wallets/:walletId/address
```
**Auth:** HMAC-only.

### Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `external_user_id` | string | No | Your internal identifier — surfaced on webhooks |
| `label` | string | No | Friendly name |
| `metadata` | object | No | Arbitrary JSON — surfaced on webhooks |
| `auto_sweep_enabled` | boolean | No | `null` = inherit from org default |
| `sweep_threshold` | string | No | Min USD value to trigger sweep on this address |

### Example

```bash
curl -X POST https://apitest.hasapay.com/api/v1/wallets/{walletId}/address \
  -H "X-API-Key: $API_KEY" \
  -H "X-Signature: $SIG" \
  -H "X-Timestamp: $TS" \
  -H "X-Request-ID: $RID" \
  -H "Content-Type: application/json" \
  -d '{
    "external_user_id": "cust_abc123",
    "label": "Customer-001",
    "metadata": {"order_id": "order_xyz"}
  }'
```

### Response

```json
{
  "data": {
    "id": "uuid",
    "master_wallet_id": "uuid",
    "organization_id": "uuid",
    "external_user_id": "cust_abc123",
    "label": "Customer-001",
    "metadata": {"order_id": "order_xyz"},
    "chain": "ethereum",
    "network": "sepolia",
    "address": "0x8ba1f109551bD432803012645Ac136ddd64DBA72",
    "public_key": "0x04...",
    "derivation_path": "m/44'/60'/0'/0/1",
    "derivation_index": 1,
    "is_active": true,
    "is_assigned": true,
    "auto_sweep_enabled": true,
    "sweep_threshold": null,
    "created_at": "2026-06-09T10:05:00Z",
    "updated_at": "2026-06-09T10:05:00Z"
  }
}
```

> Use `external_user_id` and/or `metadata` to link addresses to your internal records. Both surface on every webhook for transactions to/from this address.

---

## List addresses for one wallet

```
GET /api/v1/wallets/:walletId/addresses
```
**Auth:** Dual-auth.

### Query params

| Param | Default | Description |
|---|---|---|
| `include_balances` | `false` | Inline current balances per address |
| `limit` | `50` | Page size |
| `offset` | `0` | Pagination offset |

### Response

```json
{
  "data": {
    "addresses": [ /* ChildAddressResponse[] */ ],
    "total": 12,
    "limit": 50,
    "offset": 0
  }
}
```

---

## List all addresses (org-wide)

```
GET /api/v1/addresses
```
**Auth:** Dual-auth.

### Query params

| Param | Description |
|---|---|
| `chain` | Filter by chain |
| `network` | Filter by network |
| `include_balances` | Inline balances |
| `limit` | Default 50 |
| `offset` | Default 0 |

---

## Get one address

```
GET /api/v1/wallets/:walletId/addresses/:addressId
```
**Auth:** Dual-auth.

> The flat `GET /addresses/:id` route does **not** exist — addresses are always nested under their parent wallet.

### Query params

| Param | Default | Description |
|---|---|---|
| `include_balances` | `false` | Inline balances |

---

## Update address

```
PUT /api/v1/wallets/:walletId/addresses/:addressId
```
**Auth:** HMAC-only.

### Request body

| Field | Type | Description |
|---|---|---|
| `external_user_id` | string | New external ID |
| `label` | string | New label |
| `metadata` | object | New metadata (replaces existing) |

---

## Update auto-sweep config

```
PUT /api/v1/wallets/:walletId/addresses/:addressId/auto-sweep
```
**Auth:** HMAC-only.

### Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `auto_sweep_enabled` | boolean | Yes | Enable / disable per-address auto sweep |
| `sweep_threshold` | string | No | Per-address USD threshold; omit to clear and inherit |

For fuller sweep config (chain-level overrides, mode selection), see [Sweep](sweep.md).

---

## Send from a child address

```
POST /api/v1/wallets/:walletId/addresses/:addressId/send
```
**Auth:** HMAC-only.

See [Transactions → Send from child address](transactions.md#send-from-a-child-address) for full details.

---

## Estimate gas for a child-address send

```
POST /api/v1/wallets/:walletId/addresses/:addressId/estimate-gas
```
**Auth:** HMAC-only.

### Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `to_address` | string | Yes | Destination |
| `amount` | string | Yes | Raw token units (e.g. `100000000` for 100 USDC) |
| `asset_id` | string | No | Asset UUID; if omitted, estimates a native send |

---

## Errors

| Code | Cause |
|---|---|
| `ADDRESS_NOT_FOUND` | No address with that ID on this organization |
| `WALLET_NOT_FOUND` | Parent wallet doesn't exist or doesn't belong to caller |
| `ADDRESS_LIMIT_REACHED` | Per-wallet address cap reached (testnet has tighter limits) |
| `INVALID_METADATA` | Metadata exceeds size limit |
