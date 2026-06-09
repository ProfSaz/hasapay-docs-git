# Transactions API

View transaction history and send outgoing transactions.

**Authentication:**
- **Dual-auth (JWT or HMAC)** for all reads
- **HMAC-only** for all sends and gas estimates

**Base path:** `/api/v1/transactions`

---

## List transactions

```
GET /api/v1/transactions
```
**Auth:** Dual-auth.

### Query params

| Param | Type | Description |
|---|---|---|
| `type` | string | `deposit`, `withdrawal`, `transfer`, `sweep` |
| `status` | string | See [statuses](#statuses) below |
| `chain` | string | Filter by chain |
| `network` | string | Filter by network |
| `wallet_id` | string | Filter by master wallet ID |
| `address_id` | string | Filter by child address ID |
| `asset_id` | string | Filter by asset |
| `from_date` | string | ISO 8601 lower bound |
| `to_date` | string | ISO 8601 upper bound |
| `limit` | integer | Default 50 |
| `offset` | integer | Default 0 |

### Response

```json
{
  "data": [ /* Transaction[] */ ],
  "meta": { "limit": 50, "offset": 0, "count": 132 }
}
```

### Transaction shape

```json
{
  "id": "uuid",
  "organization_id": "uuid",
  "type": "deposit",
  "status": "confirmed",
  "chain": "ethereum",
  "network": "sepolia",
  "from_address": "0x...",
  "to_address": "0x...",
  "amount": "100.00",
  "amount_raw": "100000000",
  "tx_hash": "0xabc...",
  "block_number": 12345678,
  "confirmations": 12,
  "required_confirmations": 12,
  "asset_id": "uuid",
  "token_symbol": "USDC",
  "token_decimals": 6,
  "wallet_id": "uuid",
  "address_id": "uuid",
  "created_at": "2026-06-09T10:10:00Z",
  "updated_at": "2026-06-09T10:15:00Z"
}
```

---

## Get one transaction

```
GET /api/v1/transactions/:id
```
**Auth:** Dual-auth.

Returns the same Transaction shape as above.

---

## Get transaction status (lightweight poll)

```
GET /api/v1/transactions/:id/status
```
**Auth:** Dual-auth.

### Response

```json
{
  "data": {
    "id": "uuid",
    "status": "confirming",
    "confirmations": 3,
    "required_confirmations": 12
  }
}
```

Use this for polling — it skips the heavier joins the full GET does.

---

## Look up by on-chain hash

```
GET /api/v1/transactions/hash/:hash
```
**Auth:** Dual-auth.

Returns the Transaction matching that `tx_hash` on the caller's organization. 404 if the hash isn't recognized.

---

## Send from master wallet

```
POST /api/v1/wallets/:walletId/send
```
**Auth:** HMAC-only.

### Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `to_address` | string | Yes | Destination address |
| `amount` | string | Yes | **Raw token units** (e.g. `100000000` for 100 USDC at 6 decimals) |
| `asset_id` | string (UUID) | Yes | The asset to send — fetch from `GET /assets` |
| `idempotency_key` | string | No | Client-supplied idempotency key |

> The send body does **not** take `chain`, `network`, or `token` strings. The `asset_id` resolves chain + network + token + decimals on the server. Get asset IDs from `GET /api/v1/assets`.

### Example

```bash
curl -X POST https://apitest.hasapay.com/api/v1/wallets/{walletId}/send \
  -H "X-API-Key: $API_KEY" \
  -H "X-Signature: $SIG" \
  -H "X-Timestamp: $TS" \
  -H "X-Request-ID: $RID" \
  -H "Content-Type: application/json" \
  -d '{
    "to_address": "0x9876...",
    "amount": "50000000",
    "asset_id": "uuid-of-usdc-sepolia",
    "idempotency_key": "payout_2026_06_09_001"
  }'
```

### Response

```json
{
  "success": true,
  "message": "Transaction created and queued (sent from master wallet)",
  "data": {
    "id": "uuid",
    "type": "withdrawal",
    "status": "pending",
    "chain": "ethereum",
    "network": "sepolia",
    "from_address": "0x742d35Cc...",
    "to_address": "0x9876...",
    "amount": "50.00",
    "amount_raw": "50000000",
    "asset_id": "uuid",
    "token_symbol": "USDC",
    "created_at": "2026-06-09T11:00:00Z"
  }
}
```

The transaction begins life at `status: pending` and progresses through `broadcasted → confirming → confirmed → completed`. Listen for `withdrawal.completed` (or `.failed`) via [webhooks](webhooks.md) rather than polling.

---

## Send from a child address

```
POST /api/v1/wallets/:walletId/addresses/:addressId/send
```
**Auth:** HMAC-only.

Same request body as above (`to_address`, `amount`, `asset_id`, `idempotency_key`). Used when you want to send out of a specific child address rather than the master wallet — typically for routed-child withdrawals.

---

## Estimate gas

```
POST /api/v1/wallets/:walletId/addresses/:addressId/estimate-gas
```
**Auth:** HMAC-only.

### Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `to_address` | string | Yes | Destination |
| `amount` | string | Yes | Raw token units |
| `asset_id` | string | No | Omit for a native-token estimate |

For pure fee preview (no gas estimation, no on-chain probe) use [`GET /fees/estimate`](fees.md) instead — that walks the org's fee resolution chain and returns a usable quote without needing an address.

---

## Statuses

| Status | Meaning |
|---|---|
| `pending` | Created, awaiting broadcast (withdrawal) or first confirmation (deposit) |
| `broadcasted` | On-chain, but not yet seen by a confirmation poller |
| `confirming` | Picked up by node, confirmations accruing |
| `confirmed` | Reached `required_confirmations` |
| `completed` | Reached terminal state with all post-processing done |
| `failed` | Reverted, dropped, or expired |
| `dropped` | Evicted from mempool |
| `cancelled` | Cancelled by user (rare) |

### Typical flow

**Deposits:** `pending → confirmed → completed`
**Withdrawals:** `pending → broadcasted → confirming → confirmed → completed` (or `failed` / `dropped` at any point past `pending`)
**Sweeps:** same flow as withdrawals

---

## Types

| Type | Description |
|---|---|
| `deposit` | Incoming funds to a child address (external sender) |
| `withdrawal` | Outgoing funds sent via the API |
| `transfer` | Internal transfer between addresses (rarely surfaced) |
| `sweep` | Auto/manual sweep from child → master |

---

## Errors

| Code | Cause |
|---|---|
| `TRANSACTION_NOT_FOUND` | No transaction with that ID on this org |
| `INSUFFICIENT_BALANCE` | Source doesn't have enough funds (including gas) |
| `INVALID_ADDRESS` | Destination doesn't match the chain's address format |
| `INVALID_AMOUNT` | Amount can't be parsed as a positive integer |
| `ASSET_NOT_ENABLED` | Asset is not enabled for this org — call `POST /assets/enable` first |
| `WALLET_INACTIVE` | Wallet was deactivated |
