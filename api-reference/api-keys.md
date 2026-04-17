# API Keys

Manage API credentials for your organization.

**Authentication:** JWT

---

## List API Keys

Get all API keys for your organization.

```
GET /api/v1/api-keys
```

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/api-keys \
  -H "Authorization: Bearer {{TOKEN}}"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "api_keys": [
      {
        "id": "key_abc123",
        "name": "Production Key",
        "api_key": "hpk_live_abc123...",
        "environment": "mainnet",
        "permissions": ["wallet:read", "wallet:create", "transaction:read"],
        "is_active": true,
        "last_used_at": "2024-04-16T12:30:00Z",
        "created_at": "2024-04-16T10:00:00Z"
      }
    ]
  }
}
```

> **Note:** `secret_key` is never returned in list responses.

---

## Create API Key

Generate a new API key pair.

```
POST /api/v1/api-keys
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Friendly name |
| `environment` | string | No | `testnet` or `mainnet` |
| `permissions` | array | No | Specific permissions |

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/api-keys \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -d '{
    "name": "Production Key",
    "environment": "mainnet",
    "permissions": ["wallet:read", "wallet:create", "transaction:read", "transaction:create"]
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "key_abc123",
    "name": "Production Key",
    "api_key": "hpk_live_abc123def456789xyz",
    "secret_key": "hps_live_secretkey123456789abcdefghijklmnop",
    "environment": "mainnet",
    "permissions": ["wallet:read", "wallet:create", "transaction:read", "transaction:create"],
    "is_active": true,
    "created_at": "2024-04-16T10:00:00Z"
  }
}
```

> ⚠️ **Important:** Save the `secret_key` immediately - it's only shown once!

---

## Get API Key

Get details of a specific API key.

```
GET /api/v1/api-keys/:id
```

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/api-keys/{{API_KEY_ID}} \
  -H "Authorization: Bearer {{TOKEN}}"
```

---

## Update API Key

Update API key settings.

```
PUT /api/v1/api-keys/:id
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | New name |
| `permissions` | array | No | New permissions |

### Example Request

```bash
curl -X PUT https://apitest.hasapay.com/api/v1/api-keys/{{API_KEY_ID}} \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -d '{
    "name": "Updated Key Name",
    "permissions": ["wallet:read", "transaction:read"]
  }'
```

---

## Revoke API Key

Permanently revoke an API key.

```
DELETE /api/v1/api-keys/:id
```

### Example Request

```bash
curl -X DELETE https://apitest.hasapay.com/api/v1/api-keys/{{API_KEY_ID}} \
  -H "Authorization: Bearer {{TOKEN}}"
```

> ⚠️ **Warning:** This action is irreversible. Any requests using this key will fail immediately.

---

## Available Permissions

| Permission | Description |
|------------|-------------|
| `wallet:read` | View wallets and balances |
| `wallet:create` | Create wallets and addresses |
| `transaction:read` | View transactions |
| `transaction:create` | Send transactions |
| `webhook:read` | View webhooks |
| `webhook:create` | Manage webhooks |
| `asset:read` | View assets |
| `asset:manage` | Enable/disable assets |

---

## Errors

| Code | Description |
|------|-------------|
| `API_KEY_NOT_FOUND` | API key doesn't exist |
| `API_KEY_REVOKED` | API key has been revoked |
