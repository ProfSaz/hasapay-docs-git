# Auth Flow

Full lifecycle for the auth-related routes: register, verify, login (single + multi-org), invite handling, password reset, and password change.

**Base path:** `/api/v1/auth`

> This page is about the **auth-flow endpoints**. For the request-signing model (HMAC, JWT, dual-auth), see [Authentication](authentication.md).

---

## Register an organization

```
POST /api/v1/auth/register
```
**Auth:** Public.

Creates a new organization plus an owner user. Triggers a verification email.

### Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | Owner's full name |
| `email` | string | Yes | Owner's email — must be unique |
| `password` | string | Yes | Owner's password |
| `business_name` | string | Yes | Organization legal name |
| `website` | string | No | Org website URL |
| `incorporation_country` | string | No | ISO country code |
| `industry` | string | No | Free-form |

### Response

```json
{
  "message": "Organization created. Check your email for the verification code.",
  "data": {
    "organization_id": "uuid",
    "email": "you@company.com"
  }
}
```

---

## Verify email

```
POST /api/v1/auth/verify-email
```
**Auth:** Public.

### Request body

| Field | Type | Description |
|---|---|---|
| `organization_id` | string | From the register response |
| `code` | string | 6-digit code from the email |

---

## Resend verification code

```
POST /api/v1/auth/resend-code
```
**Auth:** Public.

| Field | Description |
|---|---|
| `organization_id` | From the register response |

---

## Login

```
POST /api/v1/auth/login
```
**Auth:** Public.

### Request body

```json
{ "email": "you@company.com", "password": "..." }
```

### Single-org response

```json
{
  "message": "Login successful",
  "data": {
    "token": "eyJhbG...",
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

Use `data.token` as your JWT — send it in `Authorization: Bearer <token>` on all JWT-tier routes.

### Multi-org response

If the user belongs to more than one organization, login does not pick one for you:

```json
{
  "message": "Please select an organization",
  "requires_org_selection": true,
  "data": {
    "session_token": "<short-lived token>",
    "organizations": [
      { "id": "uuid", "name": "Org A", "role": "owner" },
      { "id": "uuid", "name": "Org B", "role": "developer" }
    ],
    "pending_invites": []
  }
}
```

Follow up with the org selection endpoint below.

---

## Select an organization (multi-org login)

```
POST /api/v1/auth/select-org
```
**Auth:** Public (uses `session_token` from login).

### Request body

| Field | Description |
|---|---|
| `session_token` | From the multi-org login response |
| `organization_id` | UUID of the chosen org |

### Response

```json
{
  "message": "Organization selected",
  "data": {
    "token": "eyJhbG...",
    "user": { /* ... */ },
    "organization": { /* ... */ },
    "role": "owner"
  }
}
```

---

## Switch organization (already logged in)

```
POST /api/v1/auth/switch-org
```
**Auth:** JWT.

For a user who's already authenticated and wants to swap into a different org they're a member of.

| Field | Description |
|---|---|
| `organization_id` | Target org UUID |

Returns a new JWT scoped to that org.

---

## List my organizations

```
GET /api/v1/auth/my-organizations
```
**Auth:** JWT.

Returns every organization the current user belongs to, with their role in each.

---

## Forgot / reset password

### Request a reset

```
POST /api/v1/auth/forgot-password
```
**Auth:** Public.

| Field | Description |
|---|---|
| `email` | The user's email |

Sends a reset link with a token.

### Validate the token (before showing the reset form)

```
GET /api/v1/auth/reset-password/:token
```
**Auth:** Public.

Returns 200 if the token is valid and unexpired, 4xx otherwise.

### Reset the password

```
POST /api/v1/auth/reset-password/:token
```
**Auth:** Public.

| Field | Description |
|---|---|
| `password` | The new password |

---

## Change password (logged in)

```
PUT /api/v1/auth/change-password
```
**Auth:** JWT.

| Field | Description |
|---|---|
| `current_password` | Existing password |
| `new_password` | New password |

---

## Invites

The invite lifecycle has two flows depending on whether the invitee already has a HasaPay account.

### Get invite details

```
GET /api/v1/auth/invite/:token
```
**Auth:** Public.

Returns invite metadata so you can render a join page:

```json
{
  "data": {
    "email": "newperson@company.com",
    "role": "developer",
    "role_display_name": "Developer",
    "organization_id": "uuid",
    "organization_name": "Org A",
    "inviter_name": "Existing Person",
    "expires_at": "2026-06-16T10:00:00Z",
    "is_expired": false,
    "existing_user": false
  }
}
```

`existing_user` tells you which accept route to call next.

### Accept invite (new user — creates an account)

```
POST /api/v1/auth/invite/:token/accept
```
**Auth:** Public.

| Field | Description |
|---|---|
| `full_name` | Their full name |
| `password` | New account password |

Creates the user, attaches them to the inviting org, returns a JWT for that org.

### Accept invite (existing user)

```
POST /api/v1/auth/invite/:token/accept-existing
```
**Auth:** JWT.

No body. The currently-authenticated user joins the inviting org. Returns a new JWT scoped to the joined org.

### Decline invite

```
POST /api/v1/auth/invite/:token/decline
```
**Auth:** JWT.

No body. Marks the invite as declined.

### List my pending invites

```
GET /api/v1/auth/pending-invites
```
**Auth:** JWT.

Returns the invites that have been sent to the authenticated user's email and not yet accepted/declined.

---

## Auth-flow quick reference

| Flow | Calls |
|---|---|
| New org signup | `register` → email → `verify-email` → `login` |
| Owner re-login (single org) | `login` |
| Multi-org user re-login | `login` → `select-org` |
| Switch orgs mid-session | `switch-org` |
| New-user invite acceptance | invite link → `GET /auth/invite/:token` → `POST /auth/invite/:token/accept` |
| Existing-user invite acceptance | login → `GET /auth/invite/:token` → `POST /auth/invite/:token/accept-existing` |
| Forgot password | `forgot-password` → email link → `GET /auth/reset-password/:token` → `POST /auth/reset-password/:token` |
| Change password (logged in) | `PUT /auth/change-password` |
