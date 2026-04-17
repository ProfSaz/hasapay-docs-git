# Assets API

Manage supported cryptocurrencies and tokens for your organization.

**Authentication:** HMAC

---

## Get Supported Assets

Get all available assets across all chains.

```
GET /api/v1/assets/supported
```

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/assets/supported" \
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
    "chains": [
      {
        "chain": {
          "id": "chain_eth",
          "chain": "ethereum",
          "network": "sepolia",
          "display_name": "Ethereum Sepolia",
          "native_token_symbol": "ETH",
          "is_testnet": true
        },
        "assets": [
          {
            "id": "asset_eth_sepolia",
            "symbol": "ETH",
            "name": "Ethereum",
            "asset_type": "native",
            "decimals": 18,
            "is_native": true,
            "is_stablecoin": false
          },
          {
            "id": "asset_usdc_eth_sepolia",
            "symbol": "USDC",
            "name": "USD Coin",
            "asset_type": "erc20",
            "contract_address": "0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238",
            "decimals": 6,
            "is_native": false,
            "is_stablecoin": true
          }
        ]
      }
    ]
  }
}
```

---

## Get Enabled Assets

Get assets that are enabled for your organization.

```
GET /api/v1/assets
```

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/assets" \
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
    "assets": [
      {
        "enabled_asset": {
          "id": "ea_123",
          "asset_id": "asset_usdc_eth_sepolia",
          "enabled_at": "2024-04-16T10:00:00Z",
          "is_active": true
        },
        "asset": {
          "id": "asset_usdc_eth_sepolia",
          "symbol": "USDC",
          "name": "USD Coin",
          "decimals": 6
        },
        "chain": {
          "chain": "ethereum",
          "network": "sepolia"
        }
      }
    ]
  }
}
```

---

## Enable Assets

Enable one or more assets for your organization.

```
POST /api/v1/assets/enable
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `asset_ids` | array | Yes | List of asset IDs to enable |

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/assets/enable \
  -H "Content-Type: application/json" \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}" \
  -d '{
    "asset_ids": [
      "asset_eth_sepolia",
      "asset_usdc_eth_sepolia"
    ]
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "success_count": 2,
    "failed_count": 0,
    "already_enabled": []
  }
}
```

---

## Disable Asset

Disable an asset for your organization.

```
DELETE /api/v1/assets/:asset_id
```

### Example Request

```bash
curl -X DELETE https://apitest.hasapay.com/api/v1/assets/asset_usdc_eth_sepolia \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}"
```

---

## Asset Types

| Type | Description | Examples |
|------|-------------|----------|
| `native` | Native blockchain token | ETH, MATIC, TRX, BNB |
| `erc20` | Ethereum/EVM token | USDC, USDT, DAI |
| `trc20` | Tron token | USDT-TRC20 |
| `spl` | Solana token | USDC-SPL |
| `bep20` | BSC token | BUSD |

---

## Errors

| Code | Description |
|------|-------------|
| `ASSET_NOT_FOUND` | Asset doesn't exist |
| `ASSET_ALREADY_ENABLED` | Asset already enabled |
| `ASSET_NOT_ENABLED` | Asset not enabled |
