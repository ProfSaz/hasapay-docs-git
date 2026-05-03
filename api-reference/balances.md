# Balances API

Query wallet balances across all tokens.

**Authentication:** HMAC

---

## Get Single Asset Balance

Query the balance for a specific token on a wallet.

```
GET /api/v1/wallets/:walletId/balance
```

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `chain` | string | Yes | Blockchain (e.g., `ethereum`) |
| `network` | string | Yes | Network (e.g., `sepolia`) |
| `token` | string | Yes | Token symbol (e.g., `USDC`, `ETH`) |
| `token_address` | string | No | Contract address (for ERC-20/TRC-20 tokens) |

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/wallets/{{WALLET_ID}}/balance?chain=ethereum&network=sepolia&token=USDC" \
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
    "wallet_id": "550e8400-e29b-41d4-a716-446655440000",
    "token_symbol": "USDC",
    "chain": "ethereum",
    "network": "sepolia",
    "balance": "100.00",
    "balance_raw": "100000000",
    "pending_in": "50.00",
    "pending_in_raw": "50000000",
    "pending_out": "0",
    "pending_out_raw": "0",
    "decimals": 6,
    "last_updated": "2024-04-16T12:00:00Z"
  }
}
```

---

## Get All Balances

Get all token balances for a wallet.

```
GET /api/v1/wallets/:walletId/balances
```

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/wallets/{{WALLET_ID}}/balances" \
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
        "pending_in": "0",
        "pending_out": "0",
        "decimals": 18,
        "balance_usd": "1250.00",
        "is_native": true
      },
      {
        "asset_id": "asset_usdc",
        "token_symbol": "USDC",
        "token_name": "USD Coin",
        "contract_address": "0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238",
        "balance": "1000.00",
        "balance_raw": "1000000000",
        "pending_in": "50.00",
        "pending_out": "0",
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

## Balance Fields

| Field | Description |
|-------|-------------|
| `balance` | Confirmed, spendable amount |
| `pending_in` | Deposits detected but not yet confirmed |
| `pending_out` | Withdrawals submitted but not yet confirmed |

### How Balances Update

**Deposit detected (not yet confirmed):**
```
balance:      100.00 USDC
pending_in:    50.00 USDC   ← Deposit incoming
pending_out:    0.00 USDC
```

**Deposit confirmed:**
```
balance:      150.00 USDC   ← Now spendable
pending_in:     0.00 USDC
pending_out:    0.00 USDC
```

**Withdrawal initiated:**
```
balance:      100.00 USDC   ← Deducted immediately
pending_in:     0.00 USDC
pending_out:   50.00 USDC   ← Withdrawal in progress
```

**Withdrawal confirmed:**
```
balance:      100.00 USDC
pending_in:     0.00 USDC
pending_out:    0.00 USDC
```

---

## Errors

| Code | Description |
|------|-------------|
| `WALLET_NOT_FOUND` | Wallet doesn't exist |
| `INVALID_TOKEN` | Token symbol not recognized |
| `CHAIN_REQUIRED` | Chain parameter is required for single asset balance |