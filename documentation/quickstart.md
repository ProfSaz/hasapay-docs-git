# Quick Start

Get your first wallet created and receive a test deposit in under 5 minutes.

---

## Prerequisites

- A HasaPay account ([sign up](https://dashboardtest.hasapay.com/register))
- Your API key and secret key from the dashboard's API Keys page
- A REST client or shell with `curl`

---

## Step 1: Sign up and verify email

1. Register at [dashboardtest.hasapay.com/register](https://dashboardtest.hasapay.com/register)
2. Enter your organization name, email, and password
3. Check your email for a 6-digit verification code
4. Enter the code to verify

After verification, head to the API Keys page in the dashboard and generate your first key. Save the **secret key** immediately — it's only shown once.

---

## Step 2: Pick an auth tier

HasaPay has three auth tiers:

| Tier | Headers | Use when |
|---|---|---|
| **JWT** | `Authorization: Bearer <token>` | Logged in via the dashboard, hitting user/org/team/api-key routes |
| **HMAC** | `X-API-Key`, `X-Signature`, `X-Timestamp`, `X-Request-ID` | Server-to-server, hitting wallet/address writes or sends |
| **Dual-auth** | Either of the above | Every read endpoint and most config writes |

For this quickstart we'll use **HMAC** to create a wallet (HMAC-only) and read it back (dual-auth).

Full signing details in [Authentication](authentication.md).

---

## Step 3: Generate an HMAC signature

The signed payload is `{timestamp}:{requestId}:{body}` — colon-joined, no method/path.

### Node.js

```javascript
const crypto = require('crypto');
const { randomUUID } = require('crypto');

function signRequest(secretKey, body) {
  const timestamp = Math.floor(Date.now() / 1000).toString();
  const requestId = randomUUID();
  const bodyString = body ? JSON.stringify(body) : '';
  const payload = `${timestamp}:${requestId}:${bodyString}`;
  const signature = crypto
    .createHmac('sha256', secretKey)
    .update(payload)
    .digest('hex');
  return { timestamp, requestId, signature, bodyString };
}
```

### Python

```python
import hmac
import hashlib
import json
import time
import uuid

def sign_request(secret_key, body):
    timestamp = str(int(time.time()))
    request_id = str(uuid.uuid4())
    body_string = json.dumps(body, separators=(',', ':')) if body else ''
    payload = f'{timestamp}:{request_id}:{body_string}'
    signature = hmac.new(
        secret_key.encode(),
        payload.encode(),
        hashlib.sha256,
    ).hexdigest()
    return timestamp, request_id, signature, body_string
```

> Sign the bytes you send. If you serialize JSON for the signature and let your HTTP library re-serialize differently, the signatures won't match.

---

## Step 4: Create your first master wallet

```bash
BODY='{"chain":"ethereum","network":"sepolia","label":"My First Wallet"}'
TS=$(date +%s)
RID=$(uuidgen)
SIG=$(echo -n "$TS:$RID:$BODY" | openssl dgst -sha256 -hmac "$SECRET_KEY" | cut -d' ' -f2)

curl -X POST https://apitest.hasapay.com/api/v1/wallets \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -H "X-Signature: $SIG" \
  -H "X-Timestamp: $TS" \
  -H "X-Request-ID: $RID" \
  -d "$BODY"
```

### Response

```json
{
  "data": {
    "id": "uuid",
    "chain": "ethereum",
    "network": "sepolia",
    "address": "0xd502b72b8D969D60D1094174b1457C73671eb9d8",
    "label": "My First Wallet",
    "is_active": true,
    "child_count": 0,
    "created_at": "2026-06-09T10:00:00Z"
  }
}
```

🎉 You have a master wallet. Save the `id` — you'll need it for the next step.

---

## Step 5: Generate a deposit address

> **Path note:** create is `/address` (singular). Listing is `/addresses` (plural).

```bash
WALLET_ID="<from step 4>"
BODY='{"external_user_id":"cust_123","label":"Customer-001","metadata":{"order_id":"order_456"}}'
TS=$(date +%s)
RID=$(uuidgen)
SIG=$(echo -n "$TS:$RID:$BODY" | openssl dgst -sha256 -hmac "$SECRET_KEY" | cut -d' ' -f2)

curl -X POST "https://apitest.hasapay.com/api/v1/wallets/$WALLET_ID/address" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -H "X-Signature: $SIG" \
  -H "X-Timestamp: $TS" \
  -H "X-Request-ID: $RID" \
  -d "$BODY"
```

### Response

```json
{
  "data": {
    "id": "uuid",
    "master_wallet_id": "uuid",
    "address": "0x8ba1f109551bD432803012645Ac136ddd64DBA72",
    "chain": "ethereum",
    "network": "sepolia",
    "external_user_id": "cust_123",
    "metadata": {"order_id": "order_456"},
    "derivation_index": 1,
    "is_active": true,
    "auto_sweep_enabled": true,
    "created_at": "2026-06-09T10:05:00Z"
  }
}
```

The `external_user_id` and `metadata` will surface on every webhook for transactions to this address.

---

## Step 6: Get test tokens and send some in

For testnet, free faucets:

| Chain | Faucet |
|---|---|
| Ethereum Sepolia | https://sepoliafaucet.com |
| Polygon Amoy | https://faucet.polygon.technology |
| Base Sepolia | https://faucet.base.org |
| BSC Testnet | https://testnet.bnbchain.org/faucet-smart |
| Tron Shasta | https://www.trongrid.io/faucet |

Send a small amount of native token (and any test stablecoin you've enabled via `POST /assets/enable`) to your child address. Within a couple of blocks you'll see the deposit appear in the dashboard, and a `deposit.pending` → `deposit.confirmed` webhook if you've subscribed.

---

## Step 7: Set up a webhook (optional)

```bash
curl -X POST https://apitest.hasapay.com/api/v1/webhooks \
  -H "Authorization: Bearer $JWT" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-server.com/webhooks/hasapay",
    "events": ["deposit.confirmed", "withdrawal.completed", "withdrawal.failed"]
  }'
```

The webhook endpoint accepts **dual-auth**, so JWT works fine. Use your **organization's webhook secret** to verify `X-HasaPay-Signature` on incoming deliveries — see [Webhooks](../api-reference/webhooks.md).

---

## Next steps

- [Authentication](authentication.md) — full HMAC/JWT/dual-auth reference
- [API Reference Overview](../api-reference/overview.md) — every endpoint
- [Webhooks](../api-reference/webhooks.md) — payload shapes, signature verification, retries
- [Transactions](../api-reference/transactions.md) — sending and monitoring

---

## Troubleshooting

### `401 invalid_signature`

- Sign the **exact** body bytes you send on the wire — no whitespace drift between signing and sending
- Use Unix **seconds**, not milliseconds, for `X-Timestamp`
- The signed payload is `{timestamp}:{requestId}:{body}`, **not** `METHOD\nPATH\nTIMESTAMP\nBODY`

### `401 timestamp_expired`

- System clock is drifting; sync it
- Don't cache a timestamp — mint it at request time

### `409 duplicate_request`

- You reused an `X-Request-ID`. Mint a fresh UUID per request, including retries

### Still stuck?

support@hasapay.com
