# Quick Start

Get your first wallet created and receive a test deposit in under 5 minutes.

---

## Prerequisites

- A HasaPay account ([Sign up here](https://dashboardtest.hasapay.com/register))
- Your API Key and Secret Key (provided after email verification)

---

## Step 1: Register and Verify

### 1.1 Register Organization

```bash
curl -X POST https://apitest.hasapay.com/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@yourcompany.com",
    "password": "SecurePass123!",
    "business_name": "Your Company Inc",
    "website": "https://yourcompany.com",
    "incorporation_country": "US",
    "industry": "fintech"
  }'
```

### 1.2 Verify Email

Check your email for a 6-digit code, then:

```bash
curl -X POST https://apitest.hasapay.com/api/v1/auth/verify-email \
  -H "Content-Type: application/json" \
  -d '{
    "organization_id": "<from_register_response>",
    "code": "123456"
  }'
```

**Response includes your API credentials:**
```json
{
  "data": {
    "token": "eyJhbGciOi...",
    "testnet_api_key": {
      "api_key": "hpk_test_abc123...",
      "secret_key": "hps_test_xyz789..."
    }
  }
}
```

> ⚠️ **Save the secret_key immediately - it's only shown once!**

---

## Step 2: Understand Authentication

HasaPay uses **HMAC authentication** for wallet operations.

### Required Headers

| Header | Description |
|--------|-------------|
| `X-API-Key` | Your API key |
| `X-Timestamp` | Unix timestamp (seconds) |
| `X-Request-ID` | Unique UUID for each request |
| `X-Signature` | HMAC-SHA256 signature |

### Generate Signature

```javascript
const crypto = require('crypto');

const timestamp = Math.floor(Date.now() / 1000).toString();
const requestId = crypto.randomUUID();
const body = JSON.stringify({ chain: 'ethereum', network: 'sepolia', label: 'My Wallet' });

const payload = `${timestamp}:${requestId}:${body}`;
const signature = crypto.createHmac('sha256', secretKey).update(payload).digest('hex');
```

---

## Step 3: Create Your First Wallet

```bash
curl -X POST https://apitest.hasapay.com/api/v1/wallets \
  -H "Content-Type: application/json" \
  -H "X-API-Key: hpk_test_abc123..." \
  -H "X-Timestamp: 1713260400" \
  -H "X-Request-ID: 550e8400-e29b-41d4-a716-446655440000" \
  -H "X-Signature: your_generated_signature" \
  -d '{
    "chain": "ethereum",
    "network": "sepolia",
    "label": "My Test Wallet"
  }'
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "wallet_123...",
    "chain": "ethereum",
    "network": "sepolia",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f8fE00",
    "label": "My Test Wallet"
  }
}
```

---

## Step 4: Generate a Deposit Address

```bash
curl -X POST https://apitest.hasapay.com/api/v1/wallets/{wallet_id}/address \
  -H "Content-Type: application/json" \
  -H "X-API-Key: hpk_test_abc123..." \
  -H "X-Timestamp: 1713260400" \
  -H "X-Request-ID: 550e8400-e29b-41d4-a716-446655440001" \
  -H "X-Signature: your_signature" \
  -d '{
    "label": "Customer 123 Deposit",
    "auto_sweep_enabled": true
  }'
```

---

## Step 5: Get Test Tokens

For testnet, get free test tokens from faucets:

| Chain | Faucet URL |
|-------|-----------|
| Ethereum Sepolia | [sepoliafaucet.com](https://sepoliafaucet.com) |
| Polygon Amoy | [faucet.polygon.technology](https://faucet.polygon.technology) |
| Tron Shasta | [shasta.tronex.io](https://shasta.tronex.io) |
| Base Sepolia | [faucet.base.org](https://faucet.base.org) |

---

## Step 6: Monitor Transactions

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/transactions?limit=10" \
  -H "X-API-Key: hpk_test_abc123..." \
  -H "X-Timestamp: 1713260400" \
  -H "X-Request-ID: 550e8400-e29b-41d4-a716-446655440002" \
  -H "X-Signature: your_signature"
```

---

## Step 7: Set Up Webhooks

Get notified when deposits arrive:

```bash
curl -X POST https://apitest.hasapay.com/api/v1/webhooks \
  -H "Content-Type: application/json" \
  -H "X-API-Key: hpk_test_abc123..." \
  -H "X-Timestamp: 1713260400" \
  -H "X-Request-ID: 550e8400-e29b-41d4-a716-446655440003" \
  -H "X-Signature: your_signature" \
  -d '{
    "url": "https://your-server.com/webhooks/hasapay",
    "events": ["deposit.confirmed", "withdrawal.completed"],
    "secret": "your-webhook-secret"
  }'
```

---

## Next Steps

- [Authentication Deep Dive](authentication.md) - JWT and HMAC details
- [API Reference](../api-reference/overview.md) - All endpoints
- [Webhooks Guide](../api-reference/webhooks.md) - Real-time notifications
