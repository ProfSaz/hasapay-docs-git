# Webhooks API

Receive real-time notifications for deposits, withdrawals, and other events.

**Authentication:** HMAC

---

## Get Available Events

Get list of events you can subscribe to.

```
GET /api/v1/webhooks/events
```

### Example Response

```json
{
  "success": true,
  "data": {
    "events": [
      "deposit.pending",
      "deposit.confirmed",
      "withdrawal.pending",
      "withdrawal.completed",
      "withdrawal.failed",
      "sweep.completed",
      "address.created"
    ]
  }
}
```

---

## Create Webhook

Register a new webhook endpoint.

```
POST /api/v1/webhooks
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | string | Yes | Your endpoint URL (HTTPS) |
| `events` | array | Yes | Events to subscribe to |
| `secret` | string | No | Secret for signature verification |

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/webhooks \
  -H "Content-Type: application/json" \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}" \
  -d '{
    "url": "https://your-server.com/webhooks/hasapay",
    "events": ["deposit.confirmed", "withdrawal.completed"],
    "secret": "your-webhook-secret"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "990e8400-e29b-41d4-a716-446655440004",
    "url": "https://your-server.com/webhooks/hasapay",
    "events": ["deposit.confirmed", "withdrawal.completed"],
    "is_active": true,
    "created_at": "2024-04-16T10:00:00Z"
  }
}
```

---

## List Webhooks

Get all webhooks for your organization.

```
GET /api/v1/webhooks
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
        "events": ["deposit.confirmed", "withdrawal.completed"],
        "is_active": true,
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

### Example Request

```bash
curl -X PUT https://apitest.hasapay.com/api/v1/webhooks/{{WEBHOOK_ID}} \
  -H "Content-Type: application/json" \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}" \
  -d '{
    "url": "https://your-server.com/webhooks/hasapay-v2",
    "events": ["deposit.confirmed", "withdrawal.completed", "sweep.completed"],
    "is_active": true
  }'
```

---

## Delete Webhook

Remove a webhook.

```
DELETE /api/v1/webhooks/:id
```

---

## List Deliveries

Get delivery history for a webhook.

```
GET /api/v1/webhooks/:id/deliveries
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `status` | string | - | Filter: `pending`, `delivered`, `failed` |
| `limit` | integer | 50 | Items per page |

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/webhooks/{{WEBHOOK_ID}}/deliveries?status=failed&limit=50" \
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
    "deliveries": [
      {
        "id": "del_123",
        "webhook_id": "990e8400-e29b-41d4-a716-446655440004",
        "event": "deposit.confirmed",
        "status": "delivered",
        "response_code": 200,
        "attempts": 1,
        "created_at": "2024-04-16T12:00:00Z",
        "delivered_at": "2024-04-16T12:00:01Z"
      }
    ]
  }
}
```

---

## Retry Delivery

Manually retry a failed delivery.

```
POST /api/v1/webhooks/:id/deliveries/:deliveryId/retry
```

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/webhooks/{{WEBHOOK_ID}}/deliveries/{{DELIVERY_ID}}/retry \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}"
```

---

## Webhook Payload Format

When an event occurs, HasaPay sends a POST request:

### Headers

```
Content-Type: application/json
X-Webhook-Signature: sha256=abc123...
X-Webhook-Timestamp: 1713260400
```

### Body

```json
{
  "id": "evt_abc123",
  "event": "deposit.confirmed",
  "created_at": "2024-04-16T12:00:00Z",
  "data": {
    "transaction": {
      "id": "770e8400-e29b-41d4-a716-446655440002",
      "type": "deposit",
      "status": "confirmed",
      "tx_hash": "0xabc123...",
      "amount": "100.00",
      "symbol": "USDC"
    }
  }
}
```

---

## Verifying Signatures

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, timestamp, secret) {
  const signedPayload = `${timestamp}.${JSON.stringify(payload)}`;
  const expected = crypto.createHmac('sha256', secret).update(signedPayload).digest('hex');
  return `sha256=${expected}` === signature;
}
```

---

## Events

| Event | Description |
|-------|-------------|
| `deposit.pending` | Deposit detected |
| `deposit.confirmed` | Deposit confirmed |
| `withdrawal.pending` | Withdrawal submitted |
| `withdrawal.completed` | Withdrawal confirmed |
| `withdrawal.failed` | Withdrawal failed |
| `sweep.completed` | Auto-sweep completed |
| `address.created` | New address generated |

---

## Errors

| Code | Description |
|------|-------------|
| `WEBHOOK_NOT_FOUND` | Webhook doesn't exist |
| `INVALID_URL` | Invalid webhook URL |
| `DELIVERY_NOT_FOUND` | Delivery doesn't exist |
