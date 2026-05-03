# Wallets API

Create and manage HD (Hierarchical Deterministic) wallets across multiple blockchains.

**Authentication:** HMAC

**Base Path:** `/api/v1/wallets`

---

## Create Wallet

Create a new master wallet on a specific blockchain.

```
POST /api/v1/wallets
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `chain` | string | Yes | Blockchain (ethereum, polygon, tron, base, bsc, solana, bitcoin, aptos) |
| `network` | string | Yes | Network (mainnet, sepolia, amoy, shasta, etc.) |
| `label` | string | No | Friendly name for the wallet |

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/wallets \
  -H "Content-Type: application/json" \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature" \
  -d '{
    "chain": "ethereum",
    "network": "sepolia",
    "label": "Main ETH Wallet"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "chain": "ethereum",
    "network": "sepolia",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f8fE00",
    "label": "Main ETH Wallet",
    "is_active": true,
    "child_count": 0,
    "created_at": "2024-04-16T10:00:00Z",
    "updated_at": "2024-04-16T10:00:00Z"
  }
}
```

### Supported Chains & Networks

| Chain | Mainnet | Testnet |
|-------|---------|---------|
| ethereum | `mainnet` | `sepolia` |
| polygon | `mainnet` | `amoy` |
| tron | `mainnet` | `shasta` |
| base | `mainnet` | `sepolia` |
| bsc | `mainnet` | `testnet` |
| solana | `mainnet` | `devnet` |
| bitcoin | `mainnet` | `testnet` |
| aptos | `mainnet` | `testnet` |

---

## List Wallets

Get all wallets for your organization.

```
GET /api/v1/wallets
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `chain` | string | - | Filter by chain |
| `network` | string | - | Filter by network |
| `is_active` | boolean | - | Filter by active status |
| `page` | integer | 1 | Page number |
| `limit` | integer | 20 | Items per page (max 100) |

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/wallets?chain=ethereum&limit=10" \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "wallets": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "chain": "ethereum",
        "network": "sepolia",
        "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f8fE00",
        "label": "Main ETH Wallet",
        "is_active": true,
        "child_count": 5,
        "balances": [
          {
            "token_symbol": "ETH",
            "balance": "0.5",
            "balance_usd": "1250.00"
          },
          {
            "token_symbol": "USDC",
            "balance": "1000.00",
            "balance_usd": "1000.00"
          }
        ],
        "created_at": "2024-04-16T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 3,
      "total_pages": 1
    }
  }
}
```

---

## Get Wallet

Get details of a specific wallet.

```
GET /api/v1/wallets/:id
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Wallet ID (UUID) |

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/wallets/550e8400-e29b-41d4-a716-446655440000 \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "chain": "ethereum",
    "network": "sepolia",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f8fE00",
    "label": "Main ETH Wallet",
    "is_active": true,
    "child_count": 5,
    "balances": [
      {
        "asset_id": "asset_eth",
        "token_symbol": "ETH",
        "token_name": "Ethereum",
        "balance": "0.5",
        "balance_raw": "500000000000000000",
        "decimals": 18,
        "balance_usd": "1250.00"
      }
    ],
    "created_at": "2024-04-16T10:00:00Z",
    "updated_at": "2024-04-16T10:00:00Z"
  }
}
```

---

## Update Wallet

Update wallet properties.

```
PUT /api/v1/wallets/:id
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `label` | string | No | New label |
| `is_active` | boolean | No | Active status |

### Example Request

```bash
curl -X PUT https://apitest.hasapay.com/api/v1/wallets/550e8400-e29b-41d4-a716-446655440000 \
  -H "Content-Type: application/json" \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature" \
  -d '{
    "label": "Updated Wallet Name",
    "is_active": true
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "label": "Updated Wallet Name",
    "is_active": true,
    "updated_at": "2024-04-16T11:00:00Z"
  }
}
```

---

## Get Wallet Balances

Get all token balances for a wallet.

```
GET /api/v1/wallets/:id/balances
```

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/wallets/550e8400-e29b-41d4-a716-446655440000/balances \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_signature"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "wallet_id": "550e8400-e29b-41d4-a716-446655440000",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f8fE00",
    "balances": [
      {
        "asset_id": "asset_eth",
        "token_symbol": "ETH",
        "token_name": "Ethereum",
        "contract_address": null,
        "balance": "0.5",
        "balance_raw": "500000000000000000",
        "decimals": 18,
        "balance_usd": "1250.00",
        "is_native": true
      },
      {
        "asset_id": "asset_usdc",
        "token_symbol": "USDC",
        "token_name": "USD Coin",
        "contract_address": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
        "balance": "1000.00",
        "balance_raw": "1000000000",
        "decimals": 6,
        "balance_usd": "1000.00",
        "is_native": false
      }
    ],
    "total_balance_usd": "2250.00",
    "last_updated": "2024-04-16T10:30:00Z"
  }
}
```

---

## Errors

| Code | Description |
|------|-------------|
| `WALLET_NOT_FOUND` | Wallet with specified ID doesn't exist |
| `WALLET_ALREADY_EXISTS` | Wallet already exists for this chain/network |
| `INVALID_CHAIN` | Unsupported blockchain |
| `INVALID_NETWORK` | Unsupported network for this chain |
| `WALLET_LIMIT_REACHED` | Maximum wallet limit reached (testnet) |
