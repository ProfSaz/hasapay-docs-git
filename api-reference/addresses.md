# Addresses API

Generate and manage child addresses (deposit addresses) for your wallets.

**Authentication:** HMAC

---

## Create Child Address

Generate a new child address for receiving deposits.

```
POST /api/v1/wallets/:walletId/address
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `label` | string | No | Friendly name (e.g., "Customer 123") |
| `auto_sweep_enabled` | boolean | No | Enable auto-sweep to master wallet |

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/wallets/{{WALLET_ID}}/address \
  -H "Content-Type: application/json" \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}" \
  -d '{
    "label": "Customer 123 Deposit",
    "auto_sweep_enabled": true
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "wallet_id": "550e8400-e29b-41d4-a716-446655440000",
    "address": "0x8ba1f109551bD432803012645Ac136ddd64DBA72",
    "label": "Customer 123 Deposit",
    "auto_sweep_enabled": true,
    "created_at": "2024-04-16T10:05:00Z"
  }
}
```

---

## List Wallet Addresses

Get all addresses for a specific wallet.

```
GET /api/v1/wallets/:walletId/addresses
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `include_balances` | boolean | false | Include token balances |
| `limit` | integer | 50 | Items per page |
| `offset` | integer | 0 | Pagination offset |

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/wallets/{{WALLET_ID}}/addresses?include_balances=true&limit=50" \
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
    "addresses": [
      {
        "id": "660e8400-e29b-41d4-a716-446655440001",
        "wallet_id": "550e8400-e29b-41d4-a716-446655440000",
        "address": "0x8ba1f109551bD432803012645Ac136ddd64DBA72",
        "label": "Customer 123 Deposit",
        "auto_sweep_enabled": true,
        "balances": [
          {
            "symbol": "USDC",
            "balance": "100.00"
          }
        ],
        "created_at": "2024-04-16T10:05:00Z"
      }
    ],
    "pagination": {
      "limit": 50,
      "offset": 0,
      "total": 1
    }
  }
}
```

---

## List All Addresses

Get all addresses across all wallets for your organization.

```
GET /api/v1/addresses
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `chain` | string | - | Filter by chain |
| `network` | string | - | Filter by network |
| `include_balances` | boolean | false | Include token balances |

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/addresses?chain=ethereum&network=sepolia&include_balances=true" \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}"
```

---

## Get Address

Get details of a specific address.

```
GET /api/v1/wallets/:walletId/addresses/:addressId
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `include_balances` | boolean | false | Include token balances |

---

## Update Address

Update address label.

```
PUT /api/v1/wallets/:walletId/addresses/:addressId
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `label` | string | No | New label |

### Example Request

```bash
curl -X PUT https://apitest.hasapay.com/api/v1/wallets/{{WALLET_ID}}/addresses/{{ADDRESS_ID}} \
  -H "Content-Type: application/json" \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}" \
  -d '{
    "label": "Updated Label"
  }'
```

---

## Update Auto-Sweep

Configure auto-sweep settings for an address.

```
PUT /api/v1/wallets/:walletId/addresses/:addressId/auto-sweep
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable/disable auto-sweep |
| `threshold` | string | No | Minimum amount to trigger sweep |

### Example Request

```bash
curl -X PUT https://apitest.hasapay.com/api/v1/wallets/{{WALLET_ID}}/addresses/{{ADDRESS_ID}}/auto-sweep \
  -H "Content-Type: application/json" \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}" \
  -d '{
    "enabled": true,
    "threshold": "10.0"
  }'
```

---

## Errors

| Code | Description |
|------|-------------|
| `ADDRESS_NOT_FOUND` | Address doesn't exist |
| `WALLET_NOT_FOUND` | Parent wallet doesn't exist |
| `ADDRESS_LIMIT_REACHED` | Maximum addresses reached |
