# Transactions API

View transaction history and send outgoing transactions.

**Authentication:** HMAC

**Base Path:** `/api/v1/transactions`

---

## List Transactions

Get all transactions for your organization.

```
GET /api/v1/transactions
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | string | - | Filter by type: `deposit`, `withdrawal`, `transfer` |
| `status` | string | - | Filter by status: `pending`, `confirmed`, `failed` |
| `chain` | string | - | Filter by chain |
| `network` | string | - | Filter by network |
| `wallet_id` | string | - | Filter by wallet |
| `address_id` | string | - | Filter by address |
| `asset_id` | string | - | Filter by asset |
| `from_date` | string | - | Start date (ISO 8601) |
| `to_date` | string | - | End date (ISO 8601) |
| `page` | integer | 1 | Page number |
| `limit` | integer | 20 | Items per page (max 100) |

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/transactions?type=deposit&status=confirmed&limit=10" \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature"
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
        "status": "confirmed",
        "chain": "ethereum",
        "network": "sepolia",
        "wallet_id": "550e8400-e29b-41d4-a716-446655440000",
        "address_id": "660e8400-e29b-41d4-a716-446655440001",
        "from_address": "0x1234567890abcdef1234567890abcdef12345678",
        "to_address": "0x8ba1f109551bD432803012645Ac136ddd64DBA72",
        "tx_hash": "0xabc123def456...",
        "token_symbol": "USDC",
        "token_name": "USD Coin",
        "amount": "100.00",
        "amount_raw": "100000000",
        "amount_usd": "100.00",
        "fee": "0.002",
        "fee_usd": "5.00",
        "confirmations": 12,
        "required_confirmations": 12,
        "block_number": 12345678,
        "metadata": {
          "customer_id": "cust_abc123"
        },
        "confirmed_at": "2024-04-16T10:15:00Z",
        "created_at": "2024-04-16T10:10:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 25,
      "total_pages": 3
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

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Transaction ID (UUID) |

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/transactions/770e8400-e29b-41d4-a716-446655440002 \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "type": "deposit",
    "status": "confirmed",
    "chain": "ethereum",
    "network": "sepolia",
    "wallet_id": "550e8400-e29b-41d4-a716-446655440000",
    "address_id": "660e8400-e29b-41d4-a716-446655440001",
    "from_address": "0x1234567890abcdef1234567890abcdef12345678",
    "to_address": "0x8ba1f109551bD432803012645Ac136ddd64DBA72",
    "tx_hash": "0xabc123def456789abc123def456789abc123def456789abc123def456789abcd",
    "token_symbol": "USDC",
    "token_name": "USD Coin",
    "contract_address": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
    "amount": "100.00",
    "amount_raw": "100000000",
    "decimals": 6,
    "amount_usd": "100.00",
    "fee": "0.002",
    "fee_raw": "2000000000000000",
    "fee_usd": "5.00",
    "confirmations": 12,
    "required_confirmations": 12,
    "block_number": 12345678,
    "block_hash": "0xdef789...",
    "gas_used": "65000",
    "gas_price": "30000000000",
    "metadata": {
      "customer_id": "cust_abc123",
      "order_id": "order_xyz789"
    },
    "confirmed_at": "2024-04-16T10:15:00Z",
    "created_at": "2024-04-16T10:10:00Z",
    "updated_at": "2024-04-16T10:15:00Z"
  }
}
```

---

## Send Transaction

Send cryptocurrency from a wallet.

```
POST /api/v1/transactions/send
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `wallet_id` | string | Yes | Source wallet ID |
| `to_address` | string | Yes | Destination address |
| `asset_id` | string | Yes | Asset to send |
| `amount` | string | Yes | Amount to send (human readable) |
| `metadata` | object | No | Custom data to attach |
| `priority` | string | No | `low`, `medium`, `high` (affects gas) |

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/transactions/send \
  -H "Content-Type: application/json" \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature" \
  -d '{
    "wallet_id": "550e8400-e29b-41d4-a716-446655440000",
    "to_address": "0x9876543210fedcba9876543210fedcba98765432",
    "asset_id": "asset_usdc_eth_sepolia",
    "amount": "50.00",
    "metadata": {
      "payout_id": "payout_123",
      "recipient": "Vendor ABC"
    },
    "priority": "medium"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "880e8400-e29b-41d4-a716-446655440003",
    "type": "withdrawal",
    "status": "pending",
    "chain": "ethereum",
    "network": "sepolia",
    "wallet_id": "550e8400-e29b-41d4-a716-446655440000",
    "from_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f8fE00",
    "to_address": "0x9876543210fedcba9876543210fedcba98765432",
    "tx_hash": "0xpending...",
    "token_symbol": "USDC",
    "amount": "50.00",
    "amount_raw": "50000000",
    "estimated_fee": "0.003",
    "estimated_fee_usd": "7.50",
    "metadata": {
      "payout_id": "payout_123",
      "recipient": "Vendor ABC"
    },
    "created_at": "2024-04-16T11:00:00Z"
  }
}
```

---

## Transaction Statuses

| Status | Description |
|--------|-------------|
| `pending` | Transaction submitted, waiting for confirmation |
| `validating` | Transaction detected, validating on chain |
| `queued` | Transaction queued for processing |
| `confirmed` | Transaction confirmed on blockchain |
| `completed` | Transaction fully processed |
| `failed` | Transaction failed |
| `dropped` | Transaction dropped from mempool |
| `cancelled` | Transaction cancelled by user |

### Status Flow

**Deposits:**
```
validating → pending → confirmed → completed
```

**Withdrawals:**
```
queued → pending → confirmed → completed
         ↓
       failed/dropped
```

---

## Transaction Types

| Type | Description |
|------|-------------|
| `deposit` | Incoming funds to a child address |
| `withdrawal` | Outgoing funds sent via API |
| `transfer` | Internal transfer between addresses |
| `sweep` | Auto-sweep from child to master wallet |
| `fee` | Network fee transaction |

---

## Errors

| Code | Description |
|------|-------------|
| `TRANSACTION_NOT_FOUND` | Transaction with specified ID doesn't exist |
| `INSUFFICIENT_BALANCE` | Wallet doesn't have enough funds |
| `INVALID_ADDRESS` | Destination address is invalid |
| `INVALID_AMOUNT` | Amount is invalid or too small |
| `ASSET_NOT_ENABLED` | Asset is not enabled for this organization |
| `WITHDRAWAL_LIMIT_EXCEEDED` | Daily/monthly withdrawal limit exceeded |
| `WALLET_INACTIVE` | Wallet is deactivated |
