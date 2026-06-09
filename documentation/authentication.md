# Authentication

HasaPay uses three authentication tiers. Picking the right one is mostly about reading the endpoint table at the bottom of this page — but the short version is:

| Tier | Header(s) | Used for |
|---|---|---|
| **JWT** | `Authorization: Bearer <token>` | User/org/team management, API key management, settings, login flow |
| **HMAC** | `X-API-Key`, `X-Signature`, `X-Timestamp`, `X-Request-ID` | Sensitive financial writes — wallet creation, send, address management |
| **Dual-auth** | *Either* JWT *or* HMAC | Every read endpoint and most config writes — wallets/addresses/transactions/balances/fees/sweep/stats/webhooks/assets |

> If an endpoint accepts both, send whichever fits your caller — server-rendered dashboards use JWT, programmatic integrations use HMAC.

---

## JWT Authentication

JWT (JSON Web Token) authenticates requests on behalf of a logged-in user.

### Getting a JWT token

```
POST /api/v1/auth/login
```

**Request:**
```json
{
  "email": "you@company.com",
  "password": "your_password"
}
```

**Single-org user — response:**
```json
{
  "message": "Login successful",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "uuid",
      "email": "you@company.com",
      "full_name": "Your Name"
    },
    "organization": {
      "id": "uuid",
      "name": "Your Company",
      "status": "active",
      "email_verified": true
    },
    "role": "owner",
    "pending_invites": []
  }
}
```

**Multi-org user — response:**
```json
{
  "message": "Please select an organization",
  "requires_org_selection": true,
  "data": {
    "session_token": "<short-lived token>",
    "organizations": [
      { "id": "uuid", "name": "Org A", "role": "owner" }
    ],
    "pending_invites": []
  }
}
```

When `requires_org_selection` is true, follow up with `POST /api/v1/auth/select-org` (body: `{session_token, organization_id}`) to receive the final JWT.

### Using the JWT token

Send it as a Bearer header:

```bash
curl https://apitest.hasapay.com/api/v1/team/me \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

## HMAC Authentication

HMAC authenticates programmatic API key holders. The server reconstructs the signed payload from the request and compares against your `X-Signature` header.

### Required headers

| Header | Description | Example |
|---|---|---|
| `X-API-Key` | Your API key (the public half) | `WzKQ1n5L8bJ9c3VfXmnPq...` (~44 chars) |
| `X-Signature` | Hex-encoded HMAC-SHA256 of the payload | `a1b2c3...` |
| `X-Timestamp` | Unix seconds at request time | `1713260400` |
| `X-Request-ID` | A fresh UUID per request | `550e8400-e29b-41d4-a716-446655440000` |

All four are required. Missing any of them returns `401 missing_headers`.

### The signed payload

```
{timestamp}:{requestId}:{body}
```

That's a literal colon-joined string. No newlines. `{body}` is the **exact** raw JSON bytes you're sending (or empty string for GET / no body). The HTTP method and path are **not** part of the payload — only timestamp, request ID, and body matter to the signature.

**Example payload string (POST with body):**

```
1713260400:550e8400-e29b-41d4-a716-446655440000:{"name":"Production Key","permissions":["wallet:read"],"environment":"production"}
```

Sign that string with your secret key (HMAC-SHA256, hex output) and send the result as `X-Signature`.

### Timestamp window

The server accepts timestamps within **±300 seconds** (5 minutes) of its own clock. Outside that window: `401 timestamp_expired`. Generate the timestamp at request time; do not cache it.

### Replay protection

`(organization_id, X-Request-ID)` is cached server-side for `2 × window` (10 minutes). Reusing a request ID inside that window returns:

```
409 duplicate_request
```

So always mint a **fresh UUID per request** — don't reuse one across retries (a retry needs a new ID, and a new ID means a new signature because the request ID is part of the signed payload).

### Code examples

#### Node.js

```javascript
const crypto = require('crypto');
const { randomUUID } = require('crypto');
const axios = require('axios');

class HasaPayClient {
  constructor(apiKey, secretKey, baseURL = 'https://apitest.hasapay.com') {
    this.apiKey = apiKey;
    this.secretKey = secretKey;
    this.baseURL = baseURL;
  }

  sign(timestamp, requestId, body) {
    const payload = `${timestamp}:${requestId}:${body}`;
    return crypto
      .createHmac('sha256', this.secretKey)
      .update(payload)
      .digest('hex');
  }

  async request(method, path, body = null) {
    const timestamp = Math.floor(Date.now() / 1000).toString();
    const requestId = randomUUID();
    const bodyString = body ? JSON.stringify(body) : '';
    const signature = this.sign(timestamp, requestId, bodyString);

    const response = await axios({
      method,
      url: `${this.baseURL}${path}`,
      headers: {
        'Content-Type': 'application/json',
        'X-API-Key': this.apiKey,
        'X-Signature': signature,
        'X-Timestamp': timestamp,
        'X-Request-ID': requestId,
      },
      data: body,
    });
    return response.data;
  }
}

// Usage
const client = new HasaPayClient('your_api_key', 'your_secret_key');
const wallets = await client.request('GET', '/api/v1/wallets');
```

#### Python

```python
import hmac
import hashlib
import json
import time
import uuid
import requests

class HasaPayClient:
    def __init__(self, api_key, secret_key, base_url='https://apitest.hasapay.com'):
        self.api_key = api_key
        self.secret_key = secret_key
        self.base_url = base_url

    def sign(self, timestamp, request_id, body):
        payload = f'{timestamp}:{request_id}:{body}'
        return hmac.new(
            self.secret_key.encode(),
            payload.encode(),
            hashlib.sha256,
        ).hexdigest()

    def request(self, method, path, body=None):
        timestamp = str(int(time.time()))
        request_id = str(uuid.uuid4())
        body_string = json.dumps(body, separators=(',', ':')) if body else ''
        signature = self.sign(timestamp, request_id, body_string)

        return requests.request(
            method=method,
            url=f'{self.base_url}{path}',
            headers={
                'Content-Type': 'application/json',
                'X-API-Key': self.api_key,
                'X-Signature': signature,
                'X-Timestamp': timestamp,
                'X-Request-ID': request_id,
            },
            data=body_string if body else None,
        ).json()

# Usage
client = HasaPayClient('your_api_key', 'your_secret_key')
wallets = client.request('GET', '/api/v1/wallets')
```

#### Go

```go
import (
    "crypto/hmac"
    "crypto/sha256"
    "encoding/hex"
    "fmt"
    "time"
    "github.com/google/uuid"
)

func sign(secretKey, timestamp, requestID, body string) string {
    payload := fmt.Sprintf("%s:%s:%s", timestamp, requestID, body)
    mac := hmac.New(sha256.New, []byte(secretKey))
    mac.Write([]byte(payload))
    return hex.EncodeToString(mac.Sum(nil))
}

// Usage
timestamp := fmt.Sprintf("%d", time.Now().Unix())
requestID := uuid.NewString()
body := `{"chain":"ethereum","network":"sepolia"}`
signature := sign("your_secret_key", timestamp, requestID, body)
```

**Important Python note:** `json.dumps(body, separators=(',', ':'))` produces no whitespace. The bytes you send on the wire and the bytes you sign must be byte-identical — if you `json.dumps(body)` (with default separators) for signing but the request library re-serializes with different separators, the signature won't match. Easiest fix: serialize once into a string, sign that string, send that string as the body.

---

## Dual-auth endpoints

A dual-auth endpoint accepts **either** a JWT Bearer token **or** the four HMAC headers. The server figures it out from which headers are present.

Use this when:
- Building a dashboard or backend that already has a JWT in hand — just send `Authorization: Bearer ...`
- Building a server-to-server integration — sign with HMAC

You don't need to do anything special to "opt in" to dual-auth. Pick a tier and send those headers; that's it.

---

## Common HMAC errors

| Status / code | Cause | Fix |
|---|---|---|
| `401 missing_headers` | One of the four required HMAC headers is absent | Send all of `X-API-Key`, `X-Signature`, `X-Timestamp`, `X-Request-ID` |
| `401 invalid_timestamp` | `X-Timestamp` is not a parseable integer | Send Unix seconds (not milliseconds, not ISO 8601) |
| `401 timestamp_expired` | `X-Timestamp` is more than 300 seconds away from server time | Check system clock; generate timestamp at request time |
| `401 invalid_api_key` | API key is wrong, revoked, or doesn't exist | Confirm the key string + that it's active |
| `401 invalid_signature` | Signature didn't match | Most common: body bytes signed ≠ body bytes sent. Sign a string, send that exact string. |
| `409 duplicate_request` | Same `(org, X-Request-ID)` was used inside the 10-minute replay window | Mint a fresh UUID per request, including retries |

---

## Endpoint reference — which tier does each route use?

> "Dual-auth" means `RequireReadAuth` in the backend — JWT or HMAC both work.

### JWT-only

| Routes | Notes |
|---|---|
| `/auth/switch-org`, `/auth/my-organizations`, `/auth/invite/:token/accept-existing`, `/auth/invite/:token/decline`, `/auth/pending-invites`, `/auth/change-password` | Authenticated auth-flow routes |
| `/team/*` (all) | Member + invite management |
| `/settings/organization` (GET, PUT) | Org profile |
| `/api-keys/*` (all) | API key CRUD |

### HMAC-only (sensitive financial writes)

| Routes |
|---|
| `POST /wallets` |
| `POST /wallets/:walletId/address` |
| `PUT /wallets/:walletId/addresses/:addressId` |
| `PUT /wallets/:walletId/addresses/:addressId/auto-sweep` |
| `POST /wallets/:walletId/send` |
| `POST /wallets/:walletId/addresses/:addressId/send` |
| `POST /wallets/:walletId/addresses/:addressId/estimate-gas` |

### Dual-auth (JWT or HMAC)

| Category | Routes |
|---|---|
| Wallet reads | `GET /wallets`, `GET /wallets/:walletId` |
| Addresses reads | `GET /addresses`, `GET /wallets/:walletId/addresses[/:addressId]` |
| Balances | `GET /wallets/:walletId/balance`, `GET /wallets/:walletId/balances` |
| Transactions reads | `GET /transactions`, `GET /transactions/:id`, `GET /transactions/:id/status`, `GET /transactions/hash/:hash` |
| Assets | `GET /assets/supported`, `GET /assets`, `POST /assets/enable`, `DELETE /assets/:asset_id` |
| Webhooks | All `/webhooks/*` and `/deliveries` routes (incl. POST/PUT/DELETE + retry) |
| Stats | `GET /stats`, `GET /stats/volume`, `GET /stats/chains` |
| Fees | All `/fees/*` routes (config, address overrides, estimate, deposit estimate, summary, history, sources) |
| Sweep | All `/sweep/*` routes (config, per-chain overrides, addresses, history, manual trigger) |

### Public (no auth)

| Routes |
|---|
| `POST /auth/register`, `/auth/verify-email`, `/auth/resend-code` |
| `POST /auth/login`, `/auth/select-org` |
| `POST /auth/forgot-password`, `GET/POST /auth/reset-password/:token` |
| `GET /auth/invite/:token`, `POST /auth/invite/:token/accept` |

---

## Security best practices

1. **Never expose your secret key.** It's server-side only — the only time you ever see it is the create-key response. Lose it, you have to rotate.
2. **Sign the bytes you send.** Don't sign a pretty-printed JSON and send a minified one (or vice versa). Easiest invariant: serialize once into a string, sign that, send that.
3. **Generate a fresh `X-Request-ID` per request.** Reusing one is what causes `409 duplicate_request`.
4. **Use HTTPS only.** All production traffic must be TLS.
5. **Verify webhook signatures.** See [Webhooks](../api-reference/webhooks.md).
