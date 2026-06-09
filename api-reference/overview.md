# API Reference Overview

**Base URL:** `https://apitest.hasapay.com/api/v1`

---

## Authentication tiers

| Tier | Header(s) | Used for |
|---|---|---|
| **JWT** | `Authorization: Bearer <token>` | User/org/team management, API key CRUD, settings |
| **HMAC** | `X-API-Key`, `X-Signature`, `X-Timestamp`, `X-Request-ID` | Sensitive financial writes (wallet create, sends, address writes) |
| **Dual-auth** | Either JWT *or* HMAC | Every read endpoint + most config writes |

Full details in [Authentication](../documentation/authentication.md).

---

## Response shapes

### Success

Single resource:

```json
{ "data": { /* resource */ } }
```

Lists:

```json
{
  "data": [ /* resources */ ],
  "meta": { "limit": 50, "offset": 0, "count": 132 }
}
```

> Pagination uses `limit` + `offset`. There is **no** `page` / `total_pages` style.

Some endpoints (notably the create-style write routes) also include a `message` and the resource at the top level rather than under `data` — those are documented per-route.

### Error

Standard error envelope:

```json
{
  "error": {
    "code": "validation_error",
    "message": "Invalid request data",
    "details": "..."
  }
}
```

Some routes return a flat `{ "error": "..." }` (the simpler Gin form). Both are valid — check the `error` key first, treat it as the failure signal.

---

## HTTP status codes

| Code | Meaning |
|---|---|
| `200` | Success |
| `201` | Created |
| `400` | Bad request — body validation, invalid IDs |
| `401` | Unauthorized — missing/invalid auth |
| `403` | Forbidden — auth OK but caller lacks permission |
| `404` | Not found |
| `409` | Conflict — e.g. `duplicate_request` on HMAC replay |
| `422` | Unprocessable — domain validation failed |
| `429` | Rate limited |
| `500` | Server error |

---

## Rate limits

Rate limits apply per organization. Headers included on every response:

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 58
X-RateLimit-Reset: 1713260460
```

Limits scale with plan. Exceeding them returns `429`.

---

## Endpoint sections

### 🔐 Auth

User registration, login, multi-org selection, password reset, invite handling. Public + JWT-protected.

| Method | Path | Auth |
|---|---|---|
| POST | `/auth/register` | Public |
| POST | `/auth/verify-email` | Public |
| POST | `/auth/resend-code` | Public |
| POST | `/auth/login` | Public |
| POST | `/auth/select-org` | Public |
| POST | `/auth/forgot-password` | Public |
| GET | `/auth/reset-password/:token` | Public |
| POST | `/auth/reset-password/:token` | Public |
| GET | `/auth/invite/:token` | Public |
| POST | `/auth/invite/:token/accept` | Public |
| POST | `/auth/switch-org` | JWT |
| GET | `/auth/my-organizations` | JWT |
| POST | `/auth/invite/:token/accept-existing` | JWT |
| POST | `/auth/invite/:token/decline` | JWT |
| GET | `/auth/pending-invites` | JWT |
| PUT | `/auth/change-password` | JWT |

→ See [Auth flow](../documentation/auth.md)

### 👛 Wallets

| Method | Path | Auth |
|---|---|---|
| POST | `/wallets` | HMAC |
| GET | `/wallets` | Dual |
| GET | `/wallets/:walletId` | Dual |
| GET | `/wallets/:walletId/balance` | Dual |
| GET | `/wallets/:walletId/balances` | Dual |

→ See [Wallets](wallets.md)

### 📍 Addresses

| Method | Path | Auth |
|---|---|---|
| POST | `/wallets/:walletId/address` (singular!) | HMAC |
| GET | `/wallets/:walletId/addresses` | Dual |
| GET | `/wallets/:walletId/addresses/:addressId` | Dual |
| PUT | `/wallets/:walletId/addresses/:addressId` | HMAC |
| PUT | `/wallets/:walletId/addresses/:addressId/auto-sweep` | HMAC |
| GET | `/addresses` | Dual |

→ See [Addresses](addresses.md)

### 💸 Transactions

| Method | Path | Auth |
|---|---|---|
| GET | `/transactions` | Dual |
| GET | `/transactions/:id` | Dual |
| GET | `/transactions/:id/status` | Dual |
| GET | `/transactions/hash/:hash` | Dual |
| POST | `/wallets/:walletId/send` | HMAC |
| POST | `/wallets/:walletId/addresses/:addressId/send` | HMAC |
| POST | `/wallets/:walletId/addresses/:addressId/estimate-gas` | HMAC |

→ See [Transactions](transactions.md)

### 🪙 Assets

| Method | Path | Auth |
|---|---|---|
| GET | `/assets/supported` | Dual |
| GET | `/assets` | Dual |
| POST | `/assets/enable` | Dual |
| DELETE | `/assets/:asset_id` | Dual |

→ See [Assets](assets.md)

### 🔔 Webhooks

| Method | Path | Auth |
|---|---|---|
| GET | `/webhooks/events` | Dual |
| POST | `/webhooks` | Dual |
| GET | `/webhooks` | Dual |
| GET | `/webhooks/:id` | Dual |
| PUT | `/webhooks/:id` | Dual |
| DELETE | `/webhooks/:id` | Dual |
| GET | `/webhooks/:id/deliveries` | Dual |
| GET | `/webhooks/:id/deliveries/:deliveryId` | Dual |
| POST | `/webhooks/:id/deliveries/:deliveryId/retry` | Dual |
| GET | `/deliveries` | Dual |

→ See [Webhooks](webhooks.md)

### 🔑 API Keys

| Method | Path | Auth |
|---|---|---|
| GET | `/api-keys` | JWT |
| POST | `/api-keys` | JWT |
| GET | `/api-keys/:id` | JWT |
| PUT | `/api-keys/:id` | JWT |
| DELETE | `/api-keys/:id` | JWT |

→ See [API Keys](api-keys.md)

### 👥 Team

| Method | Path | Auth |
|---|---|---|
| GET | `/team/me` | JWT |
| PUT | `/team/me` | JWT |
| GET | `/team/members` | JWT |
| GET | `/team/invites` | JWT |
| POST | `/team/invite` | JWT |
| POST | `/team/invites/:id/resend` | JWT |
| DELETE | `/team/invites/:id` | JWT |
| PUT | `/team/members/:id/role` | JWT |
| POST | `/team/members/:id/deactivate` | JWT |
| POST | `/team/members/:id/reactivate` | JWT |
| DELETE | `/team/members/:id` | JWT |
| POST | `/team/leave` | JWT |

→ See [Team](team.md)

### ⚙️ Settings

| Method | Path | Auth |
|---|---|---|
| GET | `/settings/organization` | JWT |
| PUT | `/settings/organization` | JWT |

→ See [Settings](settings.md)

### 📊 Stats

| Method | Path | Auth |
|---|---|---|
| GET | `/stats` | Dual |
| GET | `/stats/volume` | Dual |
| GET | `/stats/chains` | Dual |

→ See [Stats](stats.md)

### 💰 Fees

| Method | Path | Auth |
|---|---|---|
| GET | `/fees/config` | Dual |
| GET | `/fees/addresses` | Dual |
| GET | `/fees/addresses/:addressId` | Dual |
| PUT | `/fees/addresses/:addressId` | Dual |
| DELETE | `/fees/addresses/:addressId` | Dual |
| GET | `/fees/estimate` | Dual |
| GET | `/fees/deposit/estimate` | Dual |
| GET | `/fees/summary` | Dual |
| GET | `/fees/history` | Dual |
| GET | `/fees/sources` | Dual |
| GET | `/fees/sources/:source_id` | Dual |

→ See [Fees](fees.md) *(Phase 4 doc — coming)*

### 🧹 Sweep

| Method | Path | Auth |
|---|---|---|
| GET | `/sweep/config` | Dual |
| PUT | `/sweep/config` | Dual |
| GET | `/sweep/config/:chain/:network` | Dual |
| PUT | `/sweep/config/:chain/:network` | Dual |
| DELETE | `/sweep/config/:chain/:network` | Dual |
| GET | `/sweep/addresses` | Dual |
| PUT | `/sweep/addresses/:addressId` | Dual |
| POST | `/sweep/addresses/:addressId/trigger` | Dual |
| GET | `/sweep/history` | Dual |

→ See [Sweep](sweep.md) *(Phase 4 doc — coming)*

---

## Postman collection

The most up-to-date endpoint reference outside this doc lives in the HasaPay repo: `docs/postman/orgapi.json`. Import that into Postman to test the org-facing API end-to-end.

---

## Need help?

- [Authentication](../documentation/authentication.md) — signing requests, dual-auth, JWT, HMAC, replay protection
- [Quickstart](../documentation/quickstart.md) — wallet → address → deposit in 5 minutes
- support@hasapay.com
