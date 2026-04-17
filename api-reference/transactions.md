# Transactions API

View transaction history and send outgoing transactions.

**Authentication:** HMAC

---

## Send from Master Wallet

Send cryptocurrency from a master wallet.

```
POST /api/v1/wallets/:walletId/send
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `to_address` | string | Yes | Destination address |
| `amount` | string | Yes | Amount to send |
| `asset_id` | string | Yes | Asset UUID |
| `idempotency_key` | string | No | Unique key to prevent duplicates |

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/wallets/{{WALLET_ID}}/send \
  -H "Content-Type: application/json" \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}" \
  -d '{
    "to_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb1",
    "amount": "1.5",
    "asset_id": "<asset_uuid>",
    "idempotency_key": "unique-key-123"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "type": "withdrawal",
    "status": "pending",
    "tx_hash": "0xabc123...",
    "from_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f8fE00",
    "to_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb1",
    "amount": "1.5",
    "asset_id": "<asset_uuid>",
    "created_at": "2024-04-16T11:00:00Z"
  }
}
```

---

## Send from Child Address

Send cryptocurrency from a child address.

```
POST /api/v1/wallets/:walletId/addresses/:addressId/send
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `to_address` | string | Yes | Destination address |
| `amount` | string | Yes | Amount to send |
| `asset_id` | string | Yes | Asset UUID |

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/wallets/{{WALLET_ID}}/addresses/{{ADDRESS_ID}}/send \
  -H "Content-Type: application/json" \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}" \
  -d '{
    "to_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb1",
    "amount": "1.0",
    "asset_id": "<asset_uuid>"
  }'
```

---

## Estimate Gas

Get gas estimate before sending a transaction.

```
POST /api/v1/wallets/:walletId/addresses/:addressId/estimate-gas
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `to_address` | string | Yes | Destination address |
| `amount` | string | Yes | Amount to send |
| `asset_id` | string | Yes | Asset UUID |

### Example Response

```json
{
  "success": true,
  "data": {
    "gas_limit": "21000",
    "gas_price": "30000000000",
    "estimated_fee": "0.00063",
    "estimated_fee_usd": "1.57"
  }
}
```

---

## List Transactions

Get all transactions for your organization.

```
GET /api/v1/transactions
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | string | - | Filter: `deposit`, `withdrawal`, `transfer`, `sweep` |
| `status` | string | - | Filter: `pending`, `completed`, `failed`, `broadcasted`, `confirming` |
| `chain` | string | - | Filter by chain |
| `limit` | integer | 50 | Items per page |
| `offset` | integer | 0 | Pagination offset |

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/transactions?type=deposit&status=completed&limit=50" \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "transactions": [
      {
        "id": "770e8400-e29b-41d4-a716-446655440002",
        "type": "deposit",
        "status": "completed",
        "chain": "ethereum",
        "network": "sepolia",
        "tx_hash": "0xabc123...",
        "from_address": "0x1234...",
        "to_address": "0x8ba1...",
        "amount": "100.00",
        "symbol": "USDC",
        "amount_usd": "100.00",
        "confirmations": 12,
        "created_at": "2024-04-16T10:10:00Z",
        "confirmed_at": "2024-04-16T10:15:00Z"
      }
    ],
    "pagination": {
      "limit": 50,
      "offset": 0,
      "total": 25
    }
  }
}
```

---

## Get Transaction

Get details of a specific transaction.

```
GET /api/v1/transactions/:id
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "type": "deposit",
    "status": "completed",
    "chain": "ethereum",
    "network": "sepolia",
    "wallet_id": "550e8400-e29b-41d4-a716-446655440000",
    "address_id": "660e8400-e29b-41d4-a716-446655440001",
    "tx_hash": "0xabc123def456789...",
    "block_number": 12345678,
    "from_address": "0x1234567890abcdef...",
    "to_address": "0x8ba1f109551bD432...",
    "amount": "100.00",
    "amount_raw": "100000000",
    "symbol": "USDC",
    "decimals": 6,
    "amount_usd": "100.00",
    "fee": "0.002",
    "fee_usd": "5.00",
    "confirmations": 12,
    "required_confirmations": 12,
    "created_at": "2024-04-16T10:10:00Z",
    "confirmed_at": "2024-04-16T10:15:00Z"
  }
}
```

---

## Get Transaction Status

Get just the status of a transaction.

```
GET /api/v1/transactions/:id/status
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "status": "completed",
    "confirmations": 12,
    "required_confirmations": 12
  }
}
```

---

## Get Transaction by Hash

Look up a transaction by its blockchain hash.

```
GET /api/v1/transactions/hash/:hash
```

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/transactions/hash/0xabc123def456..." \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}"
```

---

## Transaction Statuses

| Status | Description |
|--------|-------------|
| `pending` | Transaction submitted, waiting for confirmation |
| `broadcasted` | Transaction broadcast to network |
| `confirming` | Receiving confirmations |
| `completed` | Transaction confirmed |
| `failed` | Transaction failed |

---

## Transaction Types

| Type | Description |
|------|-------------|
| `deposit` | Incoming funds |
| `withdrawal` | Outgoing funds sent via API |
| `transfer` | Internal transfer |
| `sweep` | Auto-sweep from child to master |

---

## Errors

| Code | Description |
|------|-------------|
| `TRANSACTION_NOT_FOUND` | Transaction doesn't exist |
| `INSUFFICIENT_BALANCE` | Not enough funds |
| `INVALID_ADDRESS` | Invalid destination address |
| `INVALID_AMOUNT` | Invalid amount |
| `ASSET_NOT_ENABLED` | Asset not enabled |
