# Authentication

HasaPay uses two authentication methods depending on the API endpoint.

---

## Overview

| Method | Used For | Security Level |
|--------|----------|----------------|
| **JWT (Bearer Token)** | Dashboard operations, user/team management, settings, API keys | Standard |
| **HMAC (Signature)** | Wallet operations, transactions, addresses, assets, webhooks | High Security |

---

## JWT Authentication

### Login

```
POST /api/v1/auth/login
```

**Request:**
```json
{
  "email": "you@company.com",
  "password": "your_password"
}
```

**Response (Single Organization):**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "id": "user_123",
      "email": "you@company.com",
      "role": "admin"
    },
    "organization": {
      "id": "org_456",
      "name": "Your Company"
    }
  }
}
```

**Response (Multiple Organizations):**
```json
{
  "success": true,
  "data": {
    "requires_org_selection": true,
    "session_token": "sess_abc123...",
    "organizations": [
      {"id": "org_1", "name": "Company A"},
      {"id": "org_2", "name": "Company B"}
    ]
  }
}
```

### Select Organization (Multi-Org Users)

Only required when user belongs to multiple organizations.

```
POST /api/v1/auth/select-org
```

**Request:**
```json
{
  "session_token": "sess_abc123...",
  "organization_id": "org_1"
}
```

### Using the JWT Token

Include the token in the `Authorization` header:

```bash
curl -X GET https://apitest.hasapay.com/api/v1/team/me \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

---

## Registration Flow

### 1. Register Organization

```
POST /api/v1/auth/register
```

**Request:**
```json
{
  "name": "John Doe",
  "email": "john@yourcompany.com",
  "password": "SecurePass123!",
  "business_name": "Your Company Inc",
  "website": "https://yourcompany.com",
  "incorporation_country": "US",
  "industry": "fintech"
}
```

### 2. Verify Email

```
POST /api/v1/auth/verify-email
```

**Request:**
```json
{
  "organization_id": "org_456",
  "code": "123456"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "testnet_api_key": {
      "api_key": "hpk_test_abc123...",
      "secret_key": "hps_test_xyz789..."
    }
  }
}
```

> ⚠️ **Important:** Save the `secret_key` immediately - it's only shown once!

### 3. Resend Verification Code

```
POST /api/v1/auth/resend-code
```

**Request:**
```json
{
  "organization_id": "org_456",
  "email": "john@yourcompany.com"
}
```

---

## Password Management

### Change Password

Change password while logged in.

```
PUT /api/v1/auth/change-password
```

**Headers:** `Authorization: Bearer {{TOKEN}}`

**Request:**
```json
{
  "current_password": "SecurePass123!",
  "new_password": "NewSecurePass456!"
}
```

### Forgot Password

Request a password reset email.

```
POST /api/v1/auth/forgot-password
```

**Request:**
```json
{
  "email": "john@yourcompany.com"
}
```

### Validate Reset Token

```
GET /api/v1/auth/reset-password/:token
```

### Reset Password

```
POST /api/v1/auth/reset-password/:token
```

**Request:**
```json
{
  "password": "NewSecurePass123!"
}
```

---

## Organization Management

### Get My Organizations

```
GET /api/v1/auth/my-organizations
```

**Headers:** `Authorization: Bearer {{TOKEN}}`

**Response:**
```json
{
  "success": true,
  "data": {
    "organizations": [
      {"id": "org_1", "name": "Company A", "role": "admin"},
      {"id": "org_2", "name": "Company B", "role": "member"}
    ]
  }
}
```

### Switch Organization

```
POST /api/v1/auth/switch-org
```

**Headers:** `Authorization: Bearer {{TOKEN}}`

**Request:**
```json
{
  "organization_id": "org_2"
}
```

---

## Invite Flow

### Get Invite Details (Public)

```
GET /api/v1/auth/invite/:token
```

### Accept Invite (New User)

```
POST /api/v1/auth/invite/:token/accept
```

**Request:**
```json
{
  "password": "SecurePass123!"
}
```

### Accept Invite (Existing User)

```
POST /api/v1/auth/invite/:token/accept-existing
```

**Headers:** `Authorization: Bearer {{TOKEN}}`

### Decline Invite

```
POST /api/v1/auth/invite/:token/decline
```

**Headers:** `Authorization: Bearer {{TOKEN}}`

### Get Pending Invites

```
GET /api/v1/auth/pending-invites
```

**Headers:** `Authorization: Bearer {{TOKEN}}`

---

## HMAC Authentication

HMAC is used for all wallet and transaction operations.

### Required Headers

| Header | Description | Example |
|--------|-------------|---------|
| `X-API-Key` | Your API key | `hpk_test_abc123...` |
| `X-Timestamp` | Unix timestamp (seconds) | `1713260400` |
| `X-Request-ID` | Unique UUID | `550e8400-e29b-41d4...` |
| `X-Signature` | HMAC-SHA256 signature | `a1b2c3d4e5f6...` |

### Signature Generation

**Payload format:**
```
{timestamp}:{request_id}:{body}
```

### Node.js Example

```javascript
const crypto = require('crypto');

function generateHMACAuth(body, apiKey, secretKey) {
  const timestamp = Math.floor(Date.now() / 1000).toString();
  const requestId = crypto.randomUUID();
  const bodyString = body ? JSON.stringify(body) : '';

  const payload = `${timestamp}:${requestId}:${bodyString}`;
  const signature = crypto
    .createHmac('sha256', secretKey)
    .update(payload)
    .digest('hex');

  return {
    'X-API-Key': apiKey,
    'X-Timestamp': timestamp,
    'X-Request-ID': requestId,
    'X-Signature': signature
  };
}
```

### Python Example

```python
import hmac
import hashlib
import time
import uuid
import json

def generate_hmac_auth(body, api_key, secret_key):
    timestamp = str(int(time.time()))
    request_id = str(uuid.uuid4())
    body_string = json.dumps(body, separators=(',', ':')) if body else ''

    payload = f"{timestamp}:{request_id}:{body_string}"
    signature = hmac.new(
        secret_key.encode(),
        payload.encode(),
        hashlib.sha256
    ).hexdigest()

    return {
        'X-API-Key': api_key,
        'X-Timestamp': timestamp,
        'X-Request-ID': request_id,
        'X-Signature': signature
    }
```

### PHP Example

```php
<?php
function generateHMACAuth($body, $apiKey, $secretKey) {
    $timestamp = (string) time();
    $requestId = sprintf('%04x%04x-%04x-%04x-%04x-%04x%04x%04x',
        mt_rand(0, 0xffff), mt_rand(0, 0xffff),
        mt_rand(0, 0xffff),
        mt_rand(0, 0x0fff) | 0x4000,
        mt_rand(0, 0x3fff) | 0x8000,
        mt_rand(0, 0xffff), mt_rand(0, 0xffff), mt_rand(0, 0xffff)
    );
    $bodyString = $body ? json_encode($body) : '';

    $payload = "{$timestamp}:{$requestId}:{$bodyString}";
    $signature = hash_hmac('sha256', $payload, $secretKey);

    return [
        'X-API-Key' => $apiKey,
        'X-Timestamp' => $timestamp,
        'X-Request-ID' => $requestId,
        'X-Signature' => $signature
    ];
}
```

---

## Auth Endpoints Summary

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/auth/register` | None | Register organization |
| POST | `/auth/verify-email` | None | Verify email |
| POST | `/auth/resend-code` | None | Resend verification |
| POST | `/auth/login` | None | Login |
| POST | `/auth/select-org` | None | Select org (multi-org) |
| POST | `/auth/forgot-password` | None | Request password reset |
| GET | `/auth/reset-password/:token` | None | Validate reset token |
| POST | `/auth/reset-password/:token` | None | Reset password |
| GET | `/auth/invite/:token` | None | Get invite details |
| POST | `/auth/invite/:token/accept` | None | Accept invite (new user) |
| GET | `/auth/my-organizations` | JWT | List organizations |
| POST | `/auth/switch-org` | JWT | Switch organization |
| GET | `/auth/pending-invites` | JWT | List pending invites |
| POST | `/auth/invite/:token/accept-existing` | JWT | Accept invite (existing) |
| POST | `/auth/invite/:token/decline` | JWT | Decline invite |
| PUT | `/auth/change-password` | JWT | Change password |