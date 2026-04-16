# Addresses API

Generate and manage child addresses (deposit addresses) for your wallets.

**Authentication:** HMAC

**Base Path:** `/api/v1/wallets/:wallet_id/addresses` and `/api/v1/addresses`

---

## Create Address

Generate a new child address for receiving deposits.

```
POST /api/v1/wallets/:wallet_id/addresses
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `wallet_id` | string | Parent wallet ID (UUID) |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `label` | string | No | Friendly name (e.g., "Customer-123") |
| `metadata` | object | No | Custom data to attach (customer_id, order_id, etc.) |

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/wallets/550e8400-e29b-41d4-a716-446655440000/addresses \
  -H "Content-Type: application/json" \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature" \
  -d '{
    "label": "Customer-001",
    "metadata": {
      "customer_id": "cust_abc123",
      "order_id": "order_xyz789",
      "product": "Premium Plan"
    }
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
    "chain": "ethereum",
    "network": "sepolia",
    "label": "Customer-001",
    "metadata": {
      "customer_id": "cust_abc123",
      "order_id": "order_xyz789",
      "product": "Premium Plan"
    },
    "derivation_index": 1,
    "is_active": true,
    "created_at": "2024-04-16T10:05:00Z"
  }
}
```

> 💡 **Tip:** Use `metadata` to link addresses to your internal systems. When a deposit arrives, the metadata is included in the webhook payload.

---

## List Wallet Addresses

Get all addresses for a specific wallet.

```
GET /api/v1/wallets/:wallet_id/addresses
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `is_active` | boolean | - | Filter by active status |
| `page` | integer | 1 | Page number |
| `limit` | integer | 20 | Items per page (max 100) |

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/wallets/550e8400-e29b-41d4-a716-446655440000/addresses?limit=10" \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature"
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
        "label": "Customer-001",
        "metadata": {
          "customer_id": "cust_abc123"
        },
        "is_active": true,
        "created_at": "2024-04-16T10:05:00Z"
      },
      {
        "id": "660e8400-e29b-41d4-a716-446655440002",
        "wallet_id": "550e8400-e29b-41d4-a716-446655440000",
        "address": "0x9ca1f109551bD432803012645Ac136ddd64DBA73",
        "label": "Customer-002",
        "metadata": {
          "customer_id": "cust_def456"
        },
        "is_active": true,
        "created_at": "2024-04-16T10:10:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 2,
      "total_pages": 1
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
| `wallet_id` | string | - | Filter by wallet |
| `is_active` | boolean | - | Filter by active status |
| `search` | string | - | Search by label or address |
| `page` | integer | 1 | Page number |
| `limit` | integer | 20 | Items per page (max 100) |

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/addresses?chain=ethereum&search=Customer" \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature"
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
        "chain": "ethereum",
        "network": "sepolia",
        "label": "Customer-001",
        "metadata": {
          "customer_id": "cust_abc123"
        },
        "is_active": true,
        "created_at": "2024-04-16T10:05:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 1,
      "total_pages": 1
    }
  }
}
```

---

## Get Address

Get details of a specific address.

```
GET /api/v1/addresses/:id
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Address ID (UUID) |

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/addresses/660e8400-e29b-41d4-a716-446655440001 \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "wallet_id": "550e8400-e29b-41d4-a716-446655440000",
    "address": "0x8ba1f109551bD432803012645Ac136ddd64DBA72",
    "chain": "ethereum",
    "network": "sepolia",
    "label": "Customer-001",
    "metadata": {
      "customer_id": "cust_abc123",
      "order_id": "order_xyz789"
    },
    "derivation_index": 1,
    "is_active": true,
    "total_received": "1500.00",
    "total_received_usd": "1500.00",
    "transaction_count": 3,
    "created_at": "2024-04-16T10:05:00Z",
    "updated_at": "2024-04-16T12:00:00Z"
  }
}
```

---

## Update Address

Update address properties.

```
PUT /api/v1/addresses/:id
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `label` | string | No | New label |
| `metadata` | object | No | New metadata (replaces existing) |
| `is_active` | boolean | No | Active status |

### Example Request

```bash
curl -X PUT https://apitest.hasapay.com/api/v1/addresses/660e8400-e29b-41d4-a716-446655440001 \
  -H "Content-Type: application/json" \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature" \
  -d '{
    "label": "VIP Customer-001",
    "metadata": {
      "customer_id": "cust_abc123",
      "tier": "premium"
    }
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "label": "VIP Customer-001",
    "metadata": {
      "customer_id": "cust_abc123",
      "tier": "premium"
    },
    "updated_at": "2024-04-16T12:30:00Z"
  }
}
```

---

## Auto-Sweep Configuration

Configure automatic sweeping of funds from child addresses to the master wallet.

```
PUT /api/v1/addresses/:id/auto-sweep
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable/disable auto-sweep |
| `threshold` | string | No | Minimum amount to trigger sweep |
| `asset_id` | string | No | Specific asset to sweep (default: all) |

### Example Request

```bash
curl -X PUT https://apitest.hasapay.com/api/v1/addresses/660e8400-e29b-41d4-a716-446655440001/auto-sweep \
  -H "Content-Type: application/json" \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature" \
  -d '{
    "enabled": true,
    "threshold": "100.00"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "auto_sweep_enabled": true,
    "auto_sweep_threshold": "100.00",
    "updated_at": "2024-04-16T12:35:00Z"
  }
}
```

---

## Errors

| Code | Description |
|------|-------------|
| `ADDRESS_NOT_FOUND` | Address with specified ID doesn't exist |
| `WALLET_NOT_FOUND` | Parent wallet doesn't exist |
| `ADDRESS_LIMIT_REACHED` | Maximum address limit reached (testnet: 5 per wallet) |
| `INVALID_METADATA` | Metadata exceeds size limit or contains invalid data |
