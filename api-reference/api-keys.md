# API Keys

Manage HMAC API credentials for your organization.

**Authentication:** JWT
**Base path:** `/api/v1/api-keys`

> An org-scoped alias at `/api/v1/organizations/api-keys` exposes the same list/create/revoke routes — they're functionally identical, pick whichever fits your dashboard URL scheme.

---

## Overview

Each API key has two halves:

| Component | Format | Header | Description |
|---|---|---|---|
| API Key | `hpk_test_...` / `hpk_live_...` | `X-API-Key` | Public identifier |
| Secret Key | `hps_test_...` / `hps_live_...` | (used to sign, never sent) | Private key for HMAC-SHA256 |

> The secret key is **only shown once** — in the create response. Lose it, rotate the key.

---

## Create API key

```
POST /api/v1/api-keys
```

### Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | Friendly name (e.g. `"Production worker"`) |
| `permissions` | string[] | Yes | One or more from [Available permissions](#available-permissions) |
| `environment` | string | Yes | `sandbox` or `production` |

### Example

```bash
curl -X POST https://apitest.hasapay.com/api/v1/api-keys \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production worker",
    "permissions": ["wallet:read", "transaction:create", "balance:read"],
    "environment": "production"
  }'
```

### Response (201)

```json
{
  "message": "API key created successfully. Save these credentials - they won't be shown again!",
  "api_key": {
    "id": "uuid",
    "key": "hpk_live_abc123...",
    "secret_key": "hps_live_xyz789...",
    "key_prefix": "hpk_live_abc123",
    "name": "Production worker",
    "environment": "production",
    "permissions": ["wallet:read", "transaction:create", "balance:read"],
    "created_at": "2026-06-09T10:00:00Z"
  },
  "warning": "Store these credentials securely. The secret key is required for HMAC signature generation."
}
```

> The `key` and `secret_key` fields are the only place you'll ever see the full credentials. Save them immediately.

---

## List API keys

```
GET /api/v1/api-keys
```

Returns every key on the org **without** the full key string or secret — only `key_prefix` is exposed.

```json
{
  "data": [
    {
      "id": "uuid",
      "key_prefix": "hpk_live_abc123",
      "name": "Production worker",
      "permissions": ["wallet:read", "transaction:create"],
      "environment": "production",
      "is_active": true,
      "last_used_at": "2026-06-09T12:30:00Z",
      "expires_at": null,
      "created_at": "2026-06-09T10:00:00Z"
    }
  ]
}
```

---

## Get one API key

```
GET /api/v1/api-keys/:id
```

Returns the same row shape as list — never includes the secret.

---

## Update API key

```
PUT /api/v1/api-keys/:id
```

### Request body

| Field | Type | Description |
|---|---|---|
| `name` | string | New label |
| `permissions` | string[] | Replace permission set |

---

## Revoke API key

```
DELETE /api/v1/api-keys/:id
```

Revoked keys immediately reject all requests with `401 invalid_api_key`. Revocation is permanent.

---

## Available permissions

The permission strings are **singular** nouns separated by a colon — not `wallets:read`, not `read:wallet`. Send them exactly as listed:

| Permission | Grants |
|---|---|
| `wallet:read` | List + get master wallets, balances |
| `wallet:create` | Create master wallets |
| `wallet:manage` | Update master wallet settings |
| `address:read` | List + get child addresses |
| `address:create` | Create child addresses |
| `address:manage` | Update addresses, change auto-sweep |
| `balance:read` | Read balance endpoints |
| `transaction:read` | List + get transactions |
| `transaction:create` | Send transactions, estimate gas |
| `asset:read` | List supported + enabled assets |
| `asset:manage` | Enable / disable assets |
| `webhook:read` | List subscriptions + deliveries |
| `webhook:create` | Create subscriptions |
| `webhook:update` | Update subscriptions, retry deliveries |
| `webhook:delete` | Delete subscriptions |
| `fee:read` | View fee config, estimates, history, sources |
| `fee:manage` | Set / delete address fee overrides |
| `*` | Wildcard — grants every permission |

> Dashboard JWT callers go through a separate role-based permission set (owner / admin / developer / viewer) — these strings only apply to HMAC API keys.

---

## Key rotation

```javascript
// 1. Create new key
const newKey = await createApiKey({
  name: 'Production worker v2',
  permissions: existingPermissions,
  environment: 'production',
});

// 2. Update app config
process.env.HASAPAY_API_KEY = newKey.api_key.key;
process.env.HASAPAY_SECRET_KEY = newKey.api_key.secret_key;

// 3. Confirm it works
await testApiConnection();

// 4. Revoke old key
await revokeApiKey(oldKeyId);
```

---

## Errors

| Code | Cause |
|---|---|
| `API_KEY_NOT_FOUND` | No key with that ID on this org |
| `API_KEY_REVOKED` | Key has been revoked |
| `PERMISSION_DENIED` | Key lacks the permission required by the route |
| `API_KEY_LIMIT_REACHED` | Plan max reached |
