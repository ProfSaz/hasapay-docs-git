# Wallets API

Create and manage HD (Hierarchical Deterministic) wallets across multiple blockchains.

**Authentication:** HMAC

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
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}" \
  -d '{
    "chain": "ethereum",
    "network": "sepolia",
    "label": "My Test Wallet"
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
    "label": "My Test Wallet",
    "is_active": true,
    "created_at": "2024-04-16T10:00:00Z"
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
| `include_balances` | boolean | false | Include token balances |
| `chain` | string | - | Filter by chain |
| `network` | string | - | Filter by network |

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/wallets?include_balances=true" \
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
    "wallets": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "chain": "ethereum",
        "network": "sepolia",
        "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f8fE00",
        "label": "My Test Wallet",
        "is_active": true,
        "balances": [
          {
            "symbol": "ETH",
            "balance": "0.5",
            "balance_usd": "1250.00"
          },
          {
            "symbol": "USDC",
            "balance": "1000.00",
            "balance_usd": "1000.00"
          }
        ],
        "created_at": "2024-04-16T10:00:00Z"
      }
    ]
  }
}
```

---

## Get Wallet

Get details of a specific wallet.

```
GET /api/v1/wallets/:walletId
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `include_balances` | boolean | false | Include token balances |

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/wallets/{{WALLET_ID}}?include_balances=true" \
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
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "chain": "ethereum",
    "network": "sepolia",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f8fE00",
    "label": "My Test Wallet",
    "is_active": true,
    "balances": [
      {
        "asset_id": "asset_eth",
        "symbol": "ETH",
        "name": "Ethereum",
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

## Errors

| Code | Description |
|------|-------------|
| `WALLET_NOT_FOUND` | Wallet doesn't exist |
| `WALLET_ALREADY_EXISTS` | Wallet already exists for this chain/network |
| `INVALID_CHAIN` | Unsupported blockchain |
| `INVALID_NETWORK` | Unsupported network |
