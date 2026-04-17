# API Reference Overview

**Base URL:** `https://apitest.hasapay.com/api/v1`

---

## Authentication Types

| Type | Header | Used For |
|------|--------|----------|
| **JWT** | `Authorization: Bearer <token>` | Team, settings, API keys |
| **HMAC** | `X-API-Key`, `X-Timestamp`, `X-Request-ID`, `X-Signature` | Wallets, addresses, transactions, assets, webhooks |

See [Authentication](../documentation/authentication.md) for details.

---

## Response Format

### Success Response

```json
{
  "success": true,
  "data": { ... }
}
```

### Error Response

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message"
  }
}
```

### Paginated Response

```json
{
  "success": true,
  "data": {
    "items": [ ... ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 100,
      "total_pages": 5
    }
  }
}
```

---

## HTTP Status Codes

| Code | Description |
|------|-------------|
| `200` | Success |
| `201` | Created |
| `400` | Bad Request |
| `401` | Unauthorized |
| `403` | Forbidden |
| `404` | Not Found |
| `409` | Conflict |
| `422` | Validation Failed |
| `429` | Rate Limit Exceeded |
| `500` | Internal Server Error |

---

## All Endpoints

### Authentication (Public)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/register` | Register organization |
| POST | `/auth/verify-email` | Verify email |
| POST | `/auth/resend-code` | Resend verification |
| POST | `/auth/login` | Login |
| POST | `/auth/select-org` | Select org (multi-org) |
| POST | `/auth/forgot-password` | Request password reset |
| GET | `/auth/reset-password/:token` | Validate reset token |
| POST | `/auth/reset-password/:token` | Reset password |
| GET | `/auth/invite/:token` | Get invite details |
| POST | `/auth/invite/:token/accept` | Accept invite (new user) |

### Authentication (JWT)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/auth/my-organizations` | List user's organizations |
| POST | `/auth/switch-org` | Switch organization |
| GET | `/auth/pending-invites` | List pending invites |
| POST | `/auth/invite/:token/accept-existing` | Accept invite (existing user) |
| POST | `/auth/invite/:token/decline` | Decline invite |

### Wallets (HMAC)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/wallets` | Create wallet |
| GET | `/wallets` | List wallets |
| GET | `/wallets/:id` | Get wallet |

### Addresses (HMAC)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/wallets/:id/address` | Create child address |
| GET | `/wallets/:id/addresses` | List wallet addresses |
| GET | `/addresses` | List all addresses |
| GET | `/wallets/:id/addresses/:addressId` | Get address |
| PUT | `/wallets/:id/addresses/:addressId` | Update address |
| PUT | `/wallets/:id/addresses/:addressId/auto-sweep` | Update auto-sweep |

### Transactions (HMAC)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/wallets/:id/send` | Send from master wallet |
| POST | `/wallets/:id/addresses/:addressId/send` | Send from child address |
| POST | `/wallets/:id/addresses/:addressId/estimate-gas` | Estimate gas |
| GET | `/transactions` | List transactions |
| GET | `/transactions/:id` | Get transaction |
| GET | `/transactions/:id/status` | Get transaction status |
| GET | `/transactions/hash/:hash` | Get by tx hash |

### Assets (HMAC)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/assets/supported` | List supported assets |
| GET | `/assets` | List enabled assets |
| POST | `/assets/enable` | Enable assets |
| DELETE | `/assets/:id` | Disable asset |

### Webhooks (HMAC)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/webhooks/events` | List available events |
| POST | `/webhooks` | Create webhook |
| GET | `/webhooks` | List webhooks |
| GET | `/webhooks/:id` | Get webhook |
| PUT | `/webhooks/:id` | Update webhook |
| DELETE | `/webhooks/:id` | Delete webhook |
| GET | `/webhooks/:id/deliveries` | List deliveries |
| POST | `/webhooks/:id/deliveries/:deliveryId/retry` | Retry delivery |

### Team (JWT)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/team/me` | Get my profile |
| GET | `/team/members` | List team members |
| GET | `/team/invites` | List pending invites |
| POST | `/team/invite` | Invite user |
| POST | `/team/invites/:id/resend` | Resend invite |
| DELETE | `/team/invites/:id` | Cancel invite |
| PUT | `/team/members/:id/role` | Update member role |
| POST | `/team/members/:id/deactivate` | Deactivate member |
| POST | `/team/members/:id/reactivate` | Reactivate member |
| DELETE | `/team/members/:id` | Remove member |
| POST | `/team/leave` | Leave organization |

### API Keys (JWT)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api-keys` | List API keys |
| POST | `/api-keys` | Create API key |
| GET | `/api-keys/:id` | Get API key |
| PUT | `/api-keys/:id` | Update API key |
| DELETE | `/api-keys/:id` | Revoke API key |

### Settings (JWT)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/settings/organization` | Get organization settings |
| PUT | `/settings/organization` | Update organization settings |

### Stats (HMAC)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/stats` | Get organization stats |
| GET | `/stats/volume` | Get volume history |
| GET | `/stats/chains` | Get chain breakdown |

### Health (No Auth)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check |
| GET | `/ready` | Readiness check |
