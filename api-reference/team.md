# Team API

Manage team members and invitations.

**Authentication:** JWT

---

## Get My Profile

Get the current user's profile.

```
GET /api/v1/team/me
```

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/team/me \
  -H "Authorization: Bearer {{TOKEN}}"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "user_123",
    "email": "john@yourcompany.com",
    "name": "John Doe",
    "role": "admin",
    "organization": {
      "id": "org_456",
      "name": "Your Company"
    },
    "created_at": "2024-04-16T10:00:00Z"
  }
}
```

---

## List Team Members

Get all members of your organization.

```
GET /api/v1/team/members
```

### Example Response

```json
{
  "success": true,
  "data": {
    "members": [
      {
        "id": "member_123",
        "user_id": "user_123",
        "email": "john@yourcompany.com",
        "name": "John Doe",
        "role": "admin",
        "is_active": true,
        "joined_at": "2024-04-16T10:00:00Z"
      },
      {
        "id": "member_456",
        "user_id": "user_456",
        "email": "jane@yourcompany.com",
        "name": "Jane Smith",
        "role": "member",
        "is_active": true,
        "joined_at": "2024-04-17T10:00:00Z"
      }
    ]
  }
}
```

---

## List Pending Invites

Get all pending invitations.

```
GET /api/v1/team/invites
```

### Example Response

```json
{
  "success": true,
  "data": {
    "invites": [
      {
        "id": "invite_789",
        "email": "newuser@yourcompany.com",
        "name": "New User",
        "role": "member",
        "invited_by": "john@yourcompany.com",
        "created_at": "2024-04-18T10:00:00Z",
        "expires_at": "2024-04-25T10:00:00Z"
      }
    ]
  }
}
```

---

## Invite User

Send an invitation to join your organization.

```
POST /api/v1/team/invite
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `email` | string | Yes | User's email |
| `full_name` | string | Yes | User's name |
| `role` | string | Yes | Role: `admin` or `member` |

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/team/invite \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -d '{
    "email": "teammate@yourcompany.com",
    "full_name": "Team Mate",
    "role": "member"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "invite_789",
    "email": "teammate@yourcompany.com",
    "name": "Team Mate",
    "role": "member",
    "created_at": "2024-04-18T10:00:00Z"
  }
}
```

---

## Resend Invite

Resend an invitation email.

```
POST /api/v1/team/invites/:id/resend
```

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/team/invites/{{INVITE_ID}}/resend \
  -H "Authorization: Bearer {{TOKEN}}"
```

---

## Cancel Invite

Cancel a pending invitation.

```
DELETE /api/v1/team/invites/:id
```

### Example Request

```bash
curl -X DELETE https://apitest.hasapay.com/api/v1/team/invites/{{INVITE_ID}} \
  -H "Authorization: Bearer {{TOKEN}}"
```

---

## Update Member Role

Change a team member's role.

```
PUT /api/v1/team/members/:id/role
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `role` | string | Yes | New role: `admin` or `member` |

### Example Request

```bash
curl -X PUT https://apitest.hasapay.com/api/v1/team/members/{{MEMBER_ID}}/role \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -d '{
    "role": "admin"
  }'
```

---

## Deactivate Member

Temporarily deactivate a team member.

```
POST /api/v1/team/members/:id/deactivate
```

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/team/members/{{MEMBER_ID}}/deactivate \
  -H "Authorization: Bearer {{TOKEN}}"
```

---

## Reactivate Member

Reactivate a deactivated team member.

```
POST /api/v1/team/members/:id/reactivate
```

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/team/members/{{MEMBER_ID}}/reactivate \
  -H "Authorization: Bearer {{TOKEN}}"
```

---

## Remove Member

Permanently remove a team member.

```
DELETE /api/v1/team/members/:id
```

### Example Request

```bash
curl -X DELETE https://apitest.hasapay.com/api/v1/team/members/{{MEMBER_ID}} \
  -H "Authorization: Bearer {{TOKEN}}"
```

---

## Leave Organization

Leave the current organization.

```
POST /api/v1/team/leave
```

### Example Request

```bash
curl -X POST https://apitest.hasapay.com/api/v1/team/leave \
  -H "Authorization: Bearer {{TOKEN}}"
```

> ⚠️ **Warning:** You cannot leave if you're the only admin.

---

## Roles

| Role | Permissions |
|------|-------------|
| `admin` | Full access, can manage team members |
| `member` | Can use API, view data, but cannot manage team |

---

## Errors

| Code | Description |
|------|-------------|
| `MEMBER_NOT_FOUND` | Member doesn't exist |
| `INVITE_NOT_FOUND` | Invite doesn't exist |
| `INVITE_EXPIRED` | Invite has expired |
| `CANNOT_REMOVE_SELF` | Cannot remove yourself |
| `LAST_ADMIN` | Cannot leave/demote - you're the only admin |
