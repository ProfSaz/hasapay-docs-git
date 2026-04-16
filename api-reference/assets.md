# Assets API

Manage supported cryptocurrencies and tokens for your organization.

**Authentication:** JWT + HMAC (dual auth)

**Base Path:** `/api/v1/assets`

---

## Overview

HasaPay supports multiple cryptocurrencies and tokens across different blockchains. Before you can receive deposits in a specific token, you need to **enable** it for your organization.

### Asset Types

| Type | Description | Examples |
|------|-------------|----------|
| `native` | Native blockchain token | ETH, MATIC, TRX, BNB |
| `erc20` | Ethereum/EVM token | USDC, USDT, DAI |
| `trc20` | Tron token | USDT-TRC20 |
| `spl` | Solana token | USDC-SPL |
| `bep20` | BSC token | BUSD |

---

## List Supported Assets

Get all available assets across all chains.

```
GET /api/v1/assets/supported
```

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `chain` | string | Filter by chain |
| `network` | string | Filter by network |

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/assets/supported?chain=ethereum" \
  -H "Authorization: Bearer your_jwt_token"
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
          "chain_type": "evm",
          "native_token_symbol": "ETH",
          "is_testnet": true,
          "is_active": true
        },
        "assets": [
          {
            "id": "asset_eth_sepolia",
            "symbol": "ETH",
            "name": "Ethereum",
            "asset_type": "native",
            "decimals": 18,
            "is_native": true,
            "is_stablecoin": false,
            "is_verified": true,
            "enabled": false
          },
          {
            "id": "asset_usdc_eth_sepolia",
            "symbol": "USDC",
            "name": "USD Coin",
            "asset_type": "erc20",
            "contract_address": "0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238",
            "decimals": 6,
            "is_native": false,
            "is_stablecoin": true,
            "is_verified": true,
            "enabled": true
          }
        ]
      }
    ]
  }
}
```

---

## List Enabled Assets

Get assets that are enabled for your organization.

```
GET /api/v1/assets
```

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/assets \
  -H "Authorization: Bearer your_jwt_token"
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
          "organization_id": "org_456",
          "asset_id": "asset_usdc_eth_sepolia",
          "enabled_at": "2024-04-16T10:00:00Z",
          "is_active": true,
          "daily_limit": null,
          "monthly_limit": null
        },
        "asset": {
          "id": "asset_usdc_eth_sepolia",
          "symbol": "USDC",
          "name": "USD Coin",
          "asset_type": "erc20",
          "contract_address": "0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238",
          "decimals": 6,
          "is_stablecoin": true
        },
        "chain": {
          "id": "chain_eth_sepolia",
          "chain": "ethereum",
          "network": "sepolia",
          "display_name": "Ethereum Sepolia"
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
  -H "Authorization: Bearer your_jwt_token" \
  -d '{
    "asset_ids": [
      "asset_eth_sepolia",
      "asset_usdc_eth_sepolia",
      "asset_usdt_eth_sepolia"
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
    "already_enabled": ["asset_usdc_eth_sepolia"],
    "enabled": [
      {
        "asset_id": "asset_eth_sepolia",
        "symbol": "ETH",
        "chain": "ethereum"
      },
      {
        "asset_id": "asset_usdt_eth_sepolia",
        "symbol": "USDT",
        "chain": "ethereum"
      }
    ]
  }
}
```

---

## Disable Asset

Disable an asset for your organization.

```
DELETE /api/v1/assets/:id
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Asset ID |

### Example Request

```bash
curl -X DELETE https://apitest.hasapay.com/api/v1/assets/asset_usdt_eth_sepolia \
  -H "Authorization: Bearer your_jwt_token"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "message": "Asset disabled successfully"
  }
}
```

> ⚠️ **Warning:** Disabling an asset won't affect existing balances, but new deposits of that asset won't be tracked.

---

## Popular Assets by Chain

### Ethereum / EVM Chains

| Symbol | Name | Type | Contract |
|--------|------|------|----------|
| ETH | Ethereum | native | - |
| USDC | USD Coin | erc20 | 0xA0b8... |
| USDT | Tether USD | erc20 | 0xdAC1... |
| DAI | Dai Stablecoin | erc20 | 0x6B17... |

### Tron

| Symbol | Name | Type | Contract |
|--------|------|------|----------|
| TRX | Tronix | native | - |
| USDT | Tether USD | trc20 | TR7NHq... |
| USDC | USD Coin | trc20 | TEkxiT... |

### Polygon

| Symbol | Name | Type | Contract |
|--------|------|------|----------|
| MATIC | Polygon | native | - |
| USDC | USD Coin | erc20 | 0x2791... |
| USDT | Tether USD | erc20 | 0xc2132... |

### Solana

| Symbol | Name | Type | Mint Address |
|--------|------|------|--------------|
| SOL | Solana | native | - |
| USDC | USD Coin | spl | EPjFWd... |

---

## Errors

| Code | Description |
|------|-------------|
| `ASSET_NOT_FOUND` | Asset with specified ID doesn't exist |
| `ASSET_ALREADY_ENABLED` | Asset is already enabled |
| `ASSET_NOT_ENABLED` | Asset is not enabled |
| `CHAIN_NOT_SUPPORTED` | Blockchain is not supported |
| `ASSET_LIMIT_REACHED` | Maximum assets enabled (testnet limit) |
