# Team API

Manage members, roles, and invites within an organization.

**Authentication:** JWT
**Base path:** `/api/v1/team`

Role-based access control:
- `owner` — full access; only one per org
- `admin` — manage members, invites, settings; cannot remove owner
- `developer` — read everything, no member management
- `viewer` — read-only

Permission enforcement is in the service layer; calls that exceed your role return `403 forbidden`.

---

## Get current user info

```
GET /api/v1/team/me
```

Returns the membership record of the currently-authenticated user in their currently-active org.

### Response

```json
{
  "message": "User retrieved",
  "data": {
    "id": "uuid",
    "user_id": "uuid",
    "email": "you@company.com",
    "full_name": "Your Name",
    "role": "owner",
    "status": "active",
    "joined_at": "2026-06-01T10:00:00Z"
  }
}
```

---

## Update own profile

```
PUT /api/v1/team/me
```

### Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `full_name` | string | Yes | New display name |

---

## List members

```
GET /api/v1/team/members
```

Returns every active member of the organization.

### Response

```json
{
  "message": "Members retrieved",
  "data": {
    "members": [
      {
        "id": "uuid",
        "user_id": "uuid",
        "email": "owner@company.com",
        "full_name": "Owner Person",
        "role": "owner",
        "status": "active",
        "joined_at": "2026-05-01T10:00:00Z"
      }
    ],
    "count": 1
  }
}
```

---

## List pending invites

```
GET /api/v1/team/invites
```

Returns invites that have been sent and not yet accepted, declined, or expired.

```json
{
  "data": {
    "invites": [
      {
        "id": "uuid",
        "email": "newperson@company.com",
        "role": "developer",
        "role_display_name": "Developer",
        "invited_by_name": "Owner Person",
        "expires_at": "2026-06-16T10:00:00Z",
        "is_expired": false,
        "created_at": "2026-06-09T10:00:00Z"
      }
    ],
    "count": 1
  }
}
```

---

## Invite a new member

```
POST /api/v1/team/invite
```
**Role required:** `owner` or `admin`.

### Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `email` | string | Yes | Invitee's email |
| `full_name` | string | Yes | Invitee's display name |
| `role` | string | Yes | `admin`, `developer`, or `viewer` — **not** `owner` |

Returns the created invite plus emails the recipient. They follow the [invite acceptance flow](../documentation/auth.md#invites) to join.

---

## Resend invite

```
POST /api/v1/team/invites/:id/resend
```
**Role required:** `owner` or `admin`.

Re-sends the invite email and extends the expiry. No body.

---

## Cancel invite

```
DELETE /api/v1/team/invites/:id
```
**Role required:** `owner` or `admin`.

Revokes the invite. The token is immediately invalid.

---

## Update a member's role

```
PUT /api/v1/team/members/:id/role
```
**Role required:** `owner` or `admin`.

### Request body

| Field | Description |
|---|---|
| `role` | New role: `admin`, `developer`, or `viewer` |

Admins cannot promote anyone to `owner` or demote the owner. Only the current owner can transfer ownership (use a separate flow if needed).

---

## Deactivate a member

```
POST /api/v1/team/members/:id/deactivate
```
**Role required:** `owner` or `admin`.

Sets `status: deactivated`. Their JWT stops working immediately; their seat is freed for billing.

---

## Reactivate a member

```
POST /api/v1/team/members/:id/reactivate
```
**Role required:** `owner` or `admin`.

Restores a deactivated member.

---

## Remove a member

```
DELETE /api/v1/team/members/:id
```
**Role required:** `owner` or `admin`.

Permanently removes the member from the org. Different from deactivate — the membership row is deleted, not flagged.

---

## Leave organization

```
POST /api/v1/team/leave
```
**Role:** any member except `owner` (owner cannot leave; ownership must be transferred or org deleted).

The authenticated user leaves the currently-active org. Their JWT for that org is invalidated.

---

## Errors

| Code | Cause |
|---|---|
| `unauthorized` | No JWT or expired JWT |
| `forbidden` | Caller's role doesn't permit the action |
| `validation_error` | Body validation failed (bad role string, malformed email) |
| `not_found` | Member or invite not on this org |
