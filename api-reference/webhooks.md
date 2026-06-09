# Webhooks API

Receive real-time HTTP POST notifications when deposits, withdrawals, or sweeps reach a new state.

**Authentication:** Dual-auth (JWT or HMAC)
**Base path:** `/api/v1/webhooks`

---

## Supported events

| Event | Fired when |
|---|---|
| `deposit.pending` | Deposit detected on-chain, waiting for confirmations |
| `deposit.confirmed` | Deposit reached required confirmations |
| `withdrawal.initiated` | Withdrawal queued for broadcast |
| `withdrawal.completed` | Withdrawal confirmed on-chain |
| `withdrawal.failed` | Withdrawal failed (reverted, dropped, or expired) |
| `sweep.initiated` | Sweep transaction broadcast (child → master) |
| `sweep.completed` | Sweep confirmed on-chain |
| `sweep.failed` | Sweep failed (reverted, dropped, or expired) |

> The authoritative list is also available at runtime via `GET /api/v1/webhooks/events`.

---

## Create subscription

```
POST /api/v1/webhooks
```

### Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `url` | string | Yes | HTTPS endpoint to deliver to |
| `events` | string[] | Yes | One or more event names from the table above |
| `description` | string | No | Friendly label |

### Example

```bash
curl -X POST https://apitest.hasapay.com/api/v1/webhooks \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-server.com/webhooks/hasapay",
    "events": ["deposit.confirmed", "withdrawal.completed", "withdrawal.failed"],
    "description": "Production webhook"
  }'
```

### Response

```json
{
  "data": {
    "id": "990e8400-e29b-41d4-a716-446655440004",
    "url": "https://your-server.com/webhooks/hasapay",
    "events": ["deposit.confirmed", "withdrawal.completed", "withdrawal.failed"],
    "description": "Production webhook",
    "is_active": true,
    "created_at": "2026-06-09T10:00:00Z",
    "updated_at": "2026-06-09T10:00:00Z"
  }
}
```

> **No per-webhook secret is returned.** Signing uses an **org-level** webhook secret. See [Verifying signatures](#verifying-signatures) below.

---

## List subscriptions

```
GET /api/v1/webhooks
```

Returns every subscription on the organization.

---

## Get subscription

```
GET /api/v1/webhooks/:id
```

---

## Update subscription

```
PUT /api/v1/webhooks/:id
```

### Request body

Same fields as create — all optional. Send only what you want to change.

---

## Delete subscription

```
DELETE /api/v1/webhooks/:id
```

---

## List available events

```
GET /api/v1/webhooks/events
```

Returns the event list above with descriptions. Useful for building a UI that lets users pick events.

---

## Deliveries (per-subscription log)

### List deliveries for one subscription

```
GET /api/v1/webhooks/:id/deliveries?status=&limit=50
```

Query params:

| Param | Description |
|---|---|
| `status` | Filter: `pending`, `success`, `failed`, `retrying` |
| `limit` | Page size (default 50) |

### Get a single delivery

```
GET /api/v1/webhooks/:id/deliveries/:deliveryId
```

Returns the delivery row including the request payload, response status, response body, and attempt count.

### Retry a delivery

```
POST /api/v1/webhooks/:id/deliveries/:deliveryId/retry
```

Requeues the delivery for another attempt.

---

## Org-wide delivery log

```
GET /api/v1/deliveries
```

Returns deliveries across every subscription on the org. Same filters as the per-subscription list.

---

## Payload format

When an event fires, HasaPay sends a `POST` to your URL.

### Headers

| Header | Value |
|---|---|
| `Content-Type` | `application/json` |
| `User-Agent` | `HasaPay-Webhook/1.0` |
| `X-HasaPay-Event` | The event name, e.g. `deposit.confirmed` |
| `X-HasaPay-Delivery` | A UUID unique to this delivery attempt's parent delivery row |
| `X-HasaPay-Timestamp` | Unix seconds at send time |
| `X-HasaPay-Signature` | Hex HMAC-SHA256 of the raw body bytes |

### Body envelope

```json
{
  "event": "withdrawal.completed",
  "event_id": "uuid-per-delivery-per-subscription",
  "timestamp": "2026-06-09T12:00:00Z",
  "data": { /* WebhookTransactionPayload, see below */ }
}
```

`event_id` is minted **per subscription per event** — if three subscriptions are listening to the same event, each gets its own `event_id` for deduplication.

### `data` shape — `WebhookTransactionPayload`

Every transaction event uses the same `data` shape. The `from` and `to` objects are direction-aware: `master_wallet_id` and `child_address_id` are set **only** when HasaPay owns that side of the transfer.

```json
{
  "transaction_id": "uuid",
  "type": "withdrawal",
  "status": "completed",
  "chain": "ethereum",
  "network": "sepolia",
  "asset_id": "uuid",
  "token_symbol": "USDC",
  "token_decimals": 6,
  "amount": "100.00",
  "amount_raw": "100000000",
  "amount_usd": "100.00",
  "tx_hash": "0xabc...",
  "block_number": 12345678,
  "confirmations": 12,
  "required_confirmations": 12,
  "from": {
    "address": "0x742d35Cc...",
    "master_wallet_id": "uuid",
    "child_address_id": null
  },
  "to": {
    "address": "0x987654...",
    "master_wallet_id": null,
    "child_address_id": null
  },
  "fees": {
    "platform_fee_usd": "0.50",
    "org_fee_usd": "0.00",
    "gas_fee_usd": "1.20"
  },
  "metadata": { "customer_id": "cust_abc" },
  "created_at": "2026-06-09T11:55:00Z",
  "confirmed_at": "2026-06-09T12:00:00Z"
}
```

### ID placement by event type

| Type | `from.master_wallet_id` / `from.child_address_id` | `to.master_wallet_id` / `to.child_address_id` |
|---|---|---|
| `deposit` | both null (external sender) | one set (whichever side of the tx HasaPay owns) |
| `withdrawal` | one set; **both** set on routed-child withdrawals (master + child) | both null (external receiver) |
| `sweep` | `child_address_id` set | `master_wallet_id` set |

---

## Verifying signatures

The signature in `X-HasaPay-Signature` is the **hex HMAC-SHA256 of the raw request body bytes**, signed with your **organization's webhook secret**.

```
signature = hex( HMAC-SHA256( webhook_secret, raw_body_bytes ) )
```

No timestamp prefix, no JSON re-encoding, no `{ts}.{body}` concatenation. Just sign the bytes the server delivered to you. Read the raw body **before** any JSON parsing.

### Node.js (Express)

```javascript
const crypto = require('crypto');
const express = require('express');
const app = express();

const WEBHOOK_SECRET = process.env.HASAPAY_WEBHOOK_SECRET;

// Capture raw body for signature verification
app.use('/webhooks/hasapay', express.raw({ type: 'application/json' }));

app.post('/webhooks/hasapay', (req, res) => {
  const signature = req.headers['x-hasapay-signature'];
  const expected = crypto
    .createHmac('sha256', WEBHOOK_SECRET)
    .update(req.body)         // req.body is a Buffer (raw bytes)
    .digest('hex');

  if (!crypto.timingSafeEqual(Buffer.from(signature, 'hex'), Buffer.from(expected, 'hex'))) {
    return res.status(401).send('invalid signature');
  }

  const payload = JSON.parse(req.body.toString());
  switch (payload.event) {
    case 'deposit.confirmed':
      handleDeposit(payload.data);
      break;
    case 'withdrawal.completed':
      handleWithdrawal(payload.data);
      break;
  }
  res.status(200).send('ok');
});
```

### Python (Flask)

```python
import hmac
import hashlib
import json
from flask import Flask, request

app = Flask(__name__)
WEBHOOK_SECRET = os.environ['HASAPAY_WEBHOOK_SECRET']

@app.route('/webhooks/hasapay', methods=['POST'])
def webhook():
    signature = request.headers.get('X-HasaPay-Signature', '')
    expected = hmac.new(
        WEBHOOK_SECRET.encode(),
        request.get_data(),     # raw body bytes
        hashlib.sha256,
    ).hexdigest()

    if not hmac.compare_digest(signature, expected):
        return 'invalid signature', 401

    payload = json.loads(request.get_data())
    if payload['event'] == 'deposit.confirmed':
        handle_deposit(payload['data'])
    return 'ok', 200
```

> The webhook secret is per-organization, not per-subscription. You set it (or have one generated) on the organization. Two subscriptions on the same org share the same secret.

---

## Retry policy

A delivery succeeds when your endpoint returns a `2xx` status. Anything else (or a network/timeout error) is a failure, and HasaPay retries with backoff. After the max attempts, the delivery is marked `failed` and visible in the delivery log; you can manually retry it via `POST /webhooks/:id/deliveries/:deliveryId/retry`.

---

## Best practices

1. **Respond fast.** Return `200` first, process asynchronously. Slow handlers get retried.
2. **Verify signatures every time.** Anything that hit your URL without the correct `X-HasaPay-Signature` should be rejected.
3. **Deduplicate by `event_id`.** Even with retries off, you may receive the same event twice during a redeploy or transient blip.
4. **Use HTTPS.** Plaintext webhook URLs will be rejected by the create endpoint.
5. **Subscribe narrowly.** Don't subscribe to events you won't act on — every event is delivered + logged + counted.

---

## Errors

| Code | Cause |
|---|---|
| `WEBHOOK_NOT_FOUND` | No subscription with that ID on this organization |
| `INVALID_URL` | URL is not a valid HTTPS endpoint |
| `INVALID_EVENTS` | One or more event names not in the supported list |
| `WEBHOOK_LIMIT_REACHED` | Max subscriptions reached for the plan |
