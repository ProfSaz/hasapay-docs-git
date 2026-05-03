# API Keys

Manage API credentials for your organization.

**Authentication:** JWT

**Base Path:** `/api/v1/api-keys`

---

## Overview

API Keys are used to authenticate HMAC requests. Each organization can have multiple API keys for different environments or services.

### Key Components

| Component | Format | Description |
|-----------|--------|-------------|
| API Key | `hpk_...` | Public identifier, sent in `X-API-Key` header |
| Secret Key | `hps_...` | Private key for signing requests |

> ⚠️ **Security:** Secret Keys are only shown once when created. Store them securely!

---

## Create API Key

Generate a new API key pair.

```
POST /api/v1/api-keys
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Friendly name for the key |
| `environment` | string | No | `testnet` or `mainnet` (default: testnet) |
| `permissions` | array | No | Specific permissions (default: all) |
| `ip_whitelist` | array | No | Allowed IP addresses |
| `expires_at` | string | No | Expiration date (ISO 8601) |

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/api-keys \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your_jwt_token" \
  -d '{
    "name": "Production Server",
    "environment": "testnet",
    "ip_whitelist": ["203.0.113.0/24", "198.51.100.42"]
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "key_abc123def456",
    "name": "Production Server",
    "api_key": "hpk_live_abc123def456789xyz",
    "secret_key": "hps_live_secretkey123456789abcdefghijklmnop",
    "environment": "testnet",
    "permissions": ["wallets:read", "wallets:write", "transactions:read", "transactions:write"],
    "ip_whitelist": ["203.0.113.0/24", "198.51.100.42"],
    "is_active": true,
    "created_at": "2024-04-16T10:00:00Z"
  }
}
```

> ⚠️ **Important:** This is the only time `secret_key` will be shown. Save it immediately!

---

## List API Keys

Get all API keys for your organization.

```
GET /api/v1/api-keys
```

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/api-keys \
  -H "Authorization: Bearer your_jwt_token"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "api_keys": [
      {
        "id": "key_abc123def456",
        "name": "Production Server",
        "api_key": "hpk_live_abc123def456789xyz",
        "environment": "testnet",
        "permissions": ["wallets:read", "wallets:write", "transactions:read", "transactions:write"],
        "ip_whitelist": ["203.0.113.0/24"],
        "is_active": true,
        "last_used_at": "2024-04-16T12:30:00Z",
        "created_at": "2024-04-16T10:00:00Z"
      },
      {
        "id": "key_def789xyz123",
        "name": "Development",
        "api_key": "hpk_test_def789xyz123456abc",
        "environment": "testnet",
        "permissions": ["wallets:read", "transactions:read"],
        "ip_whitelist": null,
        "is_active": true,
        "last_used_at": null,
        "created_at": "2024-04-15T10:00:00Z"
      }
    ]
  }
}
```

> Note: `secret_key` is never returned in list responses.

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
| `ip_whitelist` | array | No | New IP whitelist |
| `is_active` | boolean | No | Enable/disable key |

### Example Request

```bash
curl -X PUT https://apitest.hasapay.com/api/v1/api-keys/key_abc123def456 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your_jwt_token" \
  -d '{
    "name": "Production Server v2",
    "ip_whitelist": ["203.0.113.0/24", "198.51.100.0/24"]
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "key_abc123def456",
    "name": "Production Server v2",
    "ip_whitelist": ["203.0.113.0/24", "198.51.100.0/24"],
    "updated_at": "2024-04-16T14:00:00Z"
  }
}
```

---

## Revoke API Key

Permanently revoke an API key.

```
DELETE /api/v1/api-keys/:id
```

### Example Request

```bash
curl -X DELETE https://apitest.hasapay.com/api/v1/api-keys/key_abc123def456 \
  -H "Authorization: Bearer your_jwt_token"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "message": "API key revoked successfully"
  }
}
```

> ⚠️ **Warning:** This action is irreversible. Any requests using this key will fail immediately.

---

## Available Permissions

| Permission | Description |
|------------|-------------|
| `wallets:read` | View wallets and balances |
| `wallets:write` | Create and update wallets |
| `addresses:read` | View addresses |
| `addresses:write` | Create and update addresses |
| `transactions:read` | View transactions |
| `transactions:write` | Send transactions |
| `webhooks:read` | View webhooks |
| `webhooks:write` | Create and update webhooks |
| `assets:read` | View enabled assets |
| `assets:write` | Enable/disable assets |

---

## IP Whitelisting

For enhanced security, you can restrict API key usage to specific IP addresses.

### CIDR Notation

```json
{
  "ip_whitelist": [
    "203.0.113.42",        // Single IP
    "203.0.113.0/24",      // /24 subnet (256 IPs)
    "198.51.100.0/28"      // /28 subnet (16 IPs)
  ]
}
```

### Best Practices

1. Always use IP whitelisting in production
2. Use your server's static IP, not dynamic IPs
3. Include backup IPs for failover scenarios

---

## Key Rotation

To rotate an API key:

1. Create a new API key
2. Update your application to use the new key
3. Verify the new key works correctly
4. Revoke the old key

```javascript
// Step 1: Create new key
const newKey = await createApiKey({ name: "Production v2" });

// Step 2: Update your config
process.env.HASAPAY_API_KEY = newKey.api_key;
process.env.HASAPAY_SECRET_KEY = newKey.secret_key;

// Step 3: Test
await testApiConnection();

// Step 4: Revoke old key
await revokeApiKey(oldKeyId);
```

---

## Errors

| Code | Description |
|------|-------------|
| `API_KEY_NOT_FOUND` | API key with specified ID doesn't exist |
| `API_KEY_REVOKED` | API key has been revoked |
| `API_KEY_EXPIRED` | API key has expired |
| `IP_NOT_WHITELISTED` | Request IP not in whitelist |
| `PERMISSION_DENIED` | Key doesn't have required permission |
| `API_KEY_LIMIT_REACHED` | Maximum API keys reached |
