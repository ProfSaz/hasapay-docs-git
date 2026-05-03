# Quick Start

Get your first wallet created and receive a test deposit in under 5 minutes.

---

## Prerequisites

- A HasaPay account ([Sign up here](https://dashboardtest.hasapay.com/register))
- Your API Key and Secret Key (provided after email verification)
- Basic knowledge of REST APIs

---

## Step 1: Sign Up and Verify Email

1. Go to [dashboardtest.hasapay.com/register](https://dashboardtest.hasapay.com/register)
2. Enter your organization name, email, and password
3. Check your email for a 6-digit verification code
4. Enter the code to verify your account

After verification, you'll receive an email with:
- Your **API Key** (starts with `hpk_`)
- Your **Secret Key** (starts with `hps_`)

> ⚠️ **Important:** Save your Secret Key securely. It's only shown once!

---

## Step 2: Understand Authentication

HasaPay uses two authentication methods:

| Method | Used For | Headers Required |
|--------|----------|------------------|
| **JWT** | Dashboard APIs, user management | `Authorization: Bearer <token>` |
| **HMAC** | Wallet, transaction, address APIs | `X-API-Key`, `X-Timestamp`, `X-Signature` |

For this quick start, we'll use **HMAC authentication** to create wallets.

---

## Step 3: Generate HMAC Signature

Every HMAC request requires a signature. Here's how to generate it:

### Node.js Example

```javascript
const crypto = require('crypto');

function generateSignature(method, path, timestamp, body, secretKey) {
  // Create the string to sign
  const bodyString = body ? JSON.stringify(body) : '';
  const stringToSign = `${method}\n${path}\n${timestamp}\n${bodyString}`;
  
  // Generate HMAC-SHA256 signature
  const signature = crypto
    .createHmac('sha256', secretKey)
    .update(stringToSign)
    .digest('hex');
  
  return signature;
}

// Example usage
const method = 'POST';
const path = '/api/v1/wallets';
const timestamp = Math.floor(Date.now() / 1000).toString();
const body = { chain: 'ethereum', network: 'sepolia', label: 'My Wallet' };
const secretKey = 'hps_your_secret_key';

const signature = generateSignature(method, path, timestamp, body, secretKey);
console.log(signature);
```

### Python Example

```python
import hmac
import hashlib
import json
import time

def generate_signature(method, path, timestamp, body, secret_key):
    body_string = json.dumps(body, separators=(',', ':')) if body else ''
    string_to_sign = f"{method}\n{path}\n{timestamp}\n{body_string}"
    
    signature = hmac.new(
        secret_key.encode(),
        string_to_sign.encode(),
        hashlib.sha256
    ).hexdigest()
    
    return signature

# Example usage
method = 'POST'
path = '/api/v1/wallets'
timestamp = str(int(time.time()))
body = {'chain': 'ethereum', 'network': 'sepolia', 'label': 'My Wallet'}
secret_key = 'hps_your_secret_key'

signature = generate_signature(method, path, timestamp, body, secret_key)
print(signature)
```

---

## Step 4: Create Your First Wallet

Now let's create an Ethereum testnet wallet:

### Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/wallets \
  -H "Content-Type: application/json" \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_generated_signature" \
  -d '{
    "chain": "ethereum",
    "network": "sepolia",
    "label": "My First Wallet"
  }'
```

### Response

```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "chain": "ethereum",
    "network": "sepolia",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f8fE00",
    "label": "My First Wallet",
    "is_active": true,
    "created_at": "2024-04-16T10:00:00Z"
  }
}
```

🎉 **Congratulations!** You've created your first wallet.

---

## Step 5: Generate a Deposit Address

Create a child address to receive deposits:

### Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/wallets/{wallet_id}/addresses \
  -H "Content-Type: application/json" \
  -H "X-API-Key: hpk_your_api_key" \
  -H "X-Timestamp: 1713260400" \
  -H "X-Signature: your_generated_signature" \
  -d '{
    "label": "Customer-001",
    "metadata": {
      "customer_id": "cust_123",
      "order_id": "order_456"
    }
  }'
```

### Response

```json
{
  "success": true,
  "data": {
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "wallet_id": "550e8400-e29b-41d4-a716-446655440000",
    "address": "0x8ba1f109551bD432803012645Ac136ddd64DBA72",
    "label": "Customer-001",
    "metadata": {
      "customer_id": "cust_123",
      "order_id": "order_456"
    },
    "created_at": "2024-04-16T10:05:00Z"
  }
}
```

---

## Step 6: Get Test Tokens

For testnet, you can get free test tokens from faucets:

| Chain | Faucet URL |
|-------|-----------|
| Ethereum Sepolia | [sepoliafaucet.com](https://sepoliafaucet.com) |
| Polygon Amoy | [faucet.polygon.technology](https://faucet.polygon.technology) |
| Tron Shasta | [shasta.tronex.io](https://shasta.tronex.io) |
| Base Sepolia | [faucet.base.org](https://faucet.base.org) |

Send test tokens to your generated deposit address and watch them appear in your dashboard!

---

## Step 7: Set Up Webhooks (Optional)

Get notified when deposits arrive:

### Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/webhooks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your_jwt_token" \
  -d '{
    "url": "https://your-server.com/webhook",
    "events": ["deposit.confirmed", "withdrawal.completed"],
    "is_active": true
  }'
```

---

## Next Steps

Now that you have the basics working:

1. **[Learn about Authentication](authentication.md)** - Deep dive into JWT and HMAC
2. **[Explore the API Reference](../api-reference/overview.md)** - See all available endpoints
3. **[Set up Webhooks](webhooks.md)** - Real-time notifications for deposits
4. **[View Transactions](../api-reference/transactions.md)** - Monitor all wallet activity

---

## Troubleshooting

### "Invalid signature" error

- Check that your timestamp is within 5 minutes of server time
- Ensure you're using the correct HTTP method in the signature
- Verify the path starts with `/api/v1/`
- Make sure the body JSON has no extra whitespace

### "Unauthorized" error

- Verify your API Key is correct
- Check that your account is verified
- Ensure you're using the right authentication method (JWT vs HMAC)

### Still stuck?

Contact us at support@hasapay.com or join our Discord community.
