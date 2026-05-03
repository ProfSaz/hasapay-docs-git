# Webhooks API

Receive real-time notifications for deposits, withdrawals, and other events.

**Authentication:** JWT

**Base Path:** `/api/v1/webhooks`

---

## Overview

Webhooks allow you to receive HTTP POST notifications when events occur in your HasaPay account. Instead of polling the API for updates, webhooks push data to your server instantly.

### Supported Events

| Event | Description |
|-------|-------------|
| `deposit.pending` | Deposit detected, awaiting confirmations |
| `deposit.confirmed` | Deposit confirmed on blockchain |
| `withdrawal.pending` | Withdrawal submitted to blockchain |
| `withdrawal.confirmed` | Withdrawal confirmed |
| `withdrawal.failed` | Withdrawal failed |
| `address.created` | New address generated |
| `sweep.completed` | Auto-sweep completed |

---

## Create Webhook

Register a new webhook endpoint.

```
POST /api/v1/webhooks
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | string | Yes | Your endpoint URL (must be HTTPS) |
| `events` | array | Yes | Events to subscribe to |
| `is_active` | boolean | No | Enable/disable (default: true) |
| `description` | string | No | Friendly description |

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/webhooks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your_jwt_token" \
  -d '{
    "url": "https://your-server.com/webhooks/hasapay",
    "events": ["deposit.confirmed", "withdrawal.confirmed", "withdrawal.failed"],
    "description": "Production webhook"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "990e8400-e29b-41d4-a716-446655440004",
    "url": "https://your-server.com/webhooks/hasapay",
    "events": ["deposit.confirmed", "withdrawal.confirmed", "withdrawal.failed"],
    "secret": "whsec_abc123def456xyz789...",
    "is_active": true,
    "description": "Production webhook",
    "created_at": "2024-04-16T10:00:00Z"
  }
}
```

> ⚠️ **Important:** Save the `secret` - you'll need it to verify webhook signatures. It's only shown once!

---

## List Webhooks

Get all webhooks for your organization.

```
GET /api/v1/webhooks
```

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/webhooks \
  -H "Authorization: Bearer your_jwt_token"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "webhooks": [
      {
        "id": "990e8400-e29b-41d4-a716-446655440004",
        "url": "https://your-server.com/webhooks/hasapay",
        "events": ["deposit.confirmed", "withdrawal.confirmed"],
        "is_active": true,
        "description": "Production webhook",
        "last_triggered_at": "2024-04-16T12:00:00Z",
        "success_count": 150,
        "failure_count": 2,
        "created_at": "2024-04-16T10:00:00Z"
      }
    ]
  }
}
```

---

## Get Webhook

Get details of a specific webhook.

```
GET /api/v1/webhooks/:id
```

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/webhooks/990e8400-e29b-41d4-a716-446655440004 \
  -H "Authorization: Bearer your_jwt_token"
```

---

## Update Webhook

Update webhook configuration.

```
PUT /api/v1/webhooks/:id
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | string | No | New endpoint URL |
| `events` | array | No | New events list |
| `is_active` | boolean | No | Enable/disable |
| `description` | string | No | New description |

### Example Request

```bash
curl -X PUT https://apitest.hasapay.com/api/v1/webhooks/990e8400-e29b-41d4-a716-446655440004 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your_jwt_token" \
  -d '{
    "events": ["deposit.confirmed", "deposit.pending", "withdrawal.confirmed"],
    "is_active": true
  }'
```

---

## Delete Webhook

Remove a webhook.

```
DELETE /api/v1/webhooks/:id
```

### Example Request

```bash
curl -X DELETE https://apitest.hasapay.com/api/v1/webhooks/990e8400-e29b-41d4-a716-446655440004 \
  -H "Authorization: Bearer your_jwt_token"
```

---

## Webhook Payload Format

When an event occurs, HasaPay sends a POST request to your URL with this payload:

### Headers

```
Content-Type: application/json
X-Webhook-ID: 990e8400-e29b-41d4-a716-446655440004
X-Webhook-Signature: sha256=abc123def456...
X-Webhook-Timestamp: 1713260400
```

### Body

```json
{
  "id": "evt_abc123def456",
  "event": "deposit.confirmed",
  "created_at": "2024-04-16T12:00:00Z",
  "data": {
    "transaction": {
      "id": "770e8400-e29b-41d4-a716-446655440002",
      "type": "deposit",
      "status": "confirmed",
      "chain": "ethereum",
      "network": "sepolia",
      "wallet_id": "550e8400-e29b-41d4-a716-446655440000",
      "address_id": "660e8400-e29b-41d4-a716-446655440001",
      "from_address": "0x1234...",
      "to_address": "0x8ba1...",
      "tx_hash": "0xabc123...",
      "token_symbol": "USDC",
      "amount": "100.00",
      "amount_usd": "100.00",
      "confirmations": 12,
      "metadata": {
        "customer_id": "cust_abc123",
        "order_id": "order_xyz789"
      },
      "confirmed_at": "2024-04-16T12:00:00Z"
    }
  }
}
```

---

## Verifying Webhook Signatures

Always verify webhook signatures to ensure requests are from HasaPay.

### Node.js Example

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, timestamp, secret) {
  const signedPayload = `${timestamp}.${JSON.stringify(payload)}`;
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(signedPayload)
    .digest('hex');
  
  return `sha256=${expectedSignature}` === signature;
}

// Express.js example
app.post('/webhooks/hasapay', (req, res) => {
  const signature = req.headers['x-webhook-signature'];
  const timestamp = req.headers['x-webhook-timestamp'];
  const payload = req.body;
  
  if (!verifyWebhookSignature(payload, signature, timestamp, WEBHOOK_SECRET)) {
    return res.status(401).send('Invalid signature');
  }
  
  // Process the webhook
  const { event, data } = payload;
  
  switch (event) {
    case 'deposit.confirmed':
      handleDeposit(data.transaction);
      break;
    case 'withdrawal.confirmed':
      handleWithdrawal(data.transaction);
      break;
  }
  
  res.status(200).send('OK');
});
```

### Python Example

```python
import hmac
import hashlib
from flask import Flask, request

def verify_webhook_signature(payload, signature, timestamp, secret):
    signed_payload = f"{timestamp}.{json.dumps(payload, separators=(',', ':'))}"
    expected = hmac.new(
        secret.encode(),
        signed_payload.encode(),
        hashlib.sha256
    ).hexdigest()
    return f"sha256={expected}" == signature

@app.route('/webhooks/hasapay', methods=['POST'])
def handle_webhook():
    signature = request.headers.get('X-Webhook-Signature')
    timestamp = request.headers.get('X-Webhook-Timestamp')
    payload = request.json
    
    if not verify_webhook_signature(payload, signature, timestamp, WEBHOOK_SECRET):
        return 'Invalid signature', 401
    
    event = payload['event']
    data = payload['data']
    
    if event == 'deposit.confirmed':
        handle_deposit(data['transaction'])
    
    return 'OK', 200
```

---

## Retry Policy

If your endpoint fails to respond with a `2xx` status code, HasaPay will retry:

| Attempt | Delay |
|---------|-------|
| 1 | Immediate |
| 2 | 5 seconds |
| 3 | 30 seconds |
| 4 | 2 minutes |
| 5 | 10 minutes |

After 5 failed attempts, the webhook is marked as failed. You can view failed deliveries in the dashboard.

---

## Best Practices

1. **Respond quickly** - Return `200 OK` immediately, process async
2. **Verify signatures** - Always validate the `X-Webhook-Signature`
3. **Handle duplicates** - Use the event `id` to deduplicate
4. **Use HTTPS** - Webhook URLs must use HTTPS in production
5. **Monitor failures** - Check the dashboard for failed deliveries

---

## Errors

| Code | Description |
|------|-------------|
| `WEBHOOK_NOT_FOUND` | Webhook with specified ID doesn't exist |
| `INVALID_URL` | URL is not a valid HTTPS endpoint |
| `INVALID_EVENTS` | One or more events are not valid |
| `WEBHOOK_LIMIT_REACHED` | Maximum webhook limit reached |
