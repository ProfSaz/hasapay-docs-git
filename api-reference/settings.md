# Settings API

Organization profile — name, business name, display name, timezone, contact details.

**Authentication:** JWT
**Base path:** `/api/v1/settings`

> Sweep configuration is **not** under settings — it lives at `/api/v1/sweep/config`. See [Sweep](sweep.md). Fee configuration lives at `/api/v1/fees/config` — see [Fees](fees.md).

---

## Get organization settings

```
GET /api/v1/settings/organization
```

### Response

```json
{
  "message": "Organization settings retrieved",
  "data": {
    "id": "uuid",
    "name": "Acme Inc",
    "email": "owner@acme.com",
    "email_verified": true,
    "business_name": "Acme Holdings LLC",
    "display_name": "Acme",
    "website": "https://acme.com",
    "phone": "+15555550100",
    "timezone": "America/New_York",
    "industry": "Fintech",
    "status": "active",
    "created_at": "2026-05-01T10:00:00Z"
  }
}
```

| Field | Description |
|---|---|
| `name` | Legal org name set at registration |
| `email` | Owner email — drives login + notifications |
| `email_verified` | Set once the verification code is accepted |
| `business_name` | Public-facing business name |
| `display_name` | Short form used in UI |
| `website`, `phone`, `timezone`, `industry` | Org-level metadata |
| `status` | `active`, `suspended`, etc. — set by platform admins |

---

## Update organization settings

```
PUT /api/v1/settings/organization
```
**Role required:** `owner` or `admin`.

Updates are partial — send only the fields you want to change. Omit a field to leave it untouched.

### Request body

| Field | Type | Description |
|---|---|---|
| `business_name` | string | Public business name |
| `display_name` | string | Short display name |
| `website` | string | Website URL |
| `phone` | string | Contact phone (E.164 recommended) |
| `timezone` | string | IANA tz name (`America/New_York`) |

### Example

```bash
curl -X PUT https://apitest.hasapay.com/api/v1/settings/organization \
  -H "Authorization: Bearer $JWT" \
  -H "Content-Type: application/json" \
  -d '{
    "display_name": "Acme",
    "timezone": "America/Los_Angeles"
  }'
```

### Response

```json
{
  "message": "Organization settings updated successfully",
  "data": {
    "id": "uuid",
    "business_name": "Acme Holdings LLC",
    "display_name": "Acme",
    "website": "https://acme.com",
    "phone": "+15555550100",
    "timezone": "America/Los_Angeles"
  }
}
```

### Read-only fields

These are not updatable via this endpoint:

| Field | Where to change |
|---|---|
| `name` | Contact support — legal name changes need verification |
| `email` | Auth flow (currently no in-app change) |
| `email_verified` | Verification flow — see [Auth](../documentation/auth.md) |
| `industry` | Currently set at registration only |
| `status` | Platform admin only |

---

## Errors

| Code | Cause |
|---|---|
| `unauthorized` | No JWT |
| `forbidden` | Caller is `developer` or `viewer` — only `owner`/`admin` can update |
| `validation_error` | Empty body (`No fields to update`) or invalid field shape |
| `not_found` | Org context missing — re-login |
