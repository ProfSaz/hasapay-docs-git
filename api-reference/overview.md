# API Reference Overview

Base URL: `https://apitest.hasapay.com/api/v1`

---

## Authentication Types

| Type | Header | Used For |
|------|--------|----------|
| JWT | `Authorization: Bearer <token>` | User management, webhooks, API keys |
| HMAC | `X-API-Key`, `X-Timestamp`, `X-Signature` | Wallets, addresses, transactions |

See [Authentication](../documentation/authentication.md) for details.

---

## Response Format

All responses follow this structure:

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
| `400` | Bad Request - Invalid parameters |
| `401` | Unauthorized - Invalid or missing authentication |
| `403` | Forbidden - Insufficient permissions |
| `404` | Not Found - Resource doesn't exist |
| `409` | Conflict - Resource already exists |
| `422` | Unprocessable Entity - Validation failed |
| `429` | Too Many Requests - Rate limit exceeded |
| `500` | Internal Server Error |

---

## Rate Limits

| Plan | Requests/Minute | Requests/Day |
|------|-----------------|--------------|
| Testnet | 60 | 500 |
| Starter | 120 | 10,000 |
| Growth | 300 | 50,000 |
| Enterprise | Custom | Custom |

Rate limit headers are included in every response:

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 58
X-RateLimit-Reset: 1713260460
```

---

## API Sections

### 🔐 Authentication
User registration, login, password management

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/register` | Register new organization |
| POST | `/auth/login` | Login and get JWT token |
| POST | `/auth/verify-email` | Verify email with code |
| POST | `/auth/forgot-password` | Request password reset |
| POST | `/auth/reset-password/:token` | Reset password |

### 👛 Wallets
Create and manage HD wallets

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/wallets` | Create a new wallet |
| GET | `/wallets` | List all wallets |
| GET | `/wallets/:id` | Get wallet details |
| PUT | `/wallets/:id` | Update wallet |
| GET | `/wallets/:id/balances` | Get wallet balances |

### 📍 Addresses
Generate and manage deposit addresses

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/wallets/:id/addresses` | Create child address |
| GET | `/wallets/:id/addresses` | List wallet addresses |
| GET | `/addresses` | List all addresses |
| GET | `/addresses/:id` | Get address details |
| PUT | `/addresses/:id` | Update address |

### 💸 Transactions
View and send transactions

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/transactions` | List all transactions |
| GET | `/transactions/:id` | Get transaction details |
| POST | `/transactions/send` | Send a transaction |

### 🪙 Assets
Manage supported cryptocurrencies

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/assets/supported` | List all available assets |
| GET | `/assets` | List enabled assets |
| POST | `/assets/enable` | Enable assets |
| DELETE | `/assets/:id` | Disable an asset |

### 🔔 Webhooks
Real-time event notifications

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/webhooks` | Create webhook |
| GET | `/webhooks` | List webhooks |
| GET | `/webhooks/:id` | Get webhook details |
| PUT | `/webhooks/:id` | Update webhook |
| DELETE | `/webhooks/:id` | Delete webhook |

### 🔑 API Keys
Manage API credentials

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api-keys` | Create API key |
| GET | `/api-keys` | List API keys |
| PUT | `/api-keys/:id` | Update API key |
| DELETE | `/api-keys/:id` | Revoke API key |

### 👥 Team
Team member management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/team/me` | Get current user |
| PUT | `/team/me` | Update profile |
| GET | `/team/members` | List team members |
| POST | `/team/invite` | Invite team member |
| DELETE | `/team/members/:id` | Remove team member |

### 📊 Stats
Dashboard statistics

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/stats` | Get organization stats |
| GET | `/stats/volume` | Get volume history |
| GET | `/stats/chains` | Get chain breakdown |

---

## Postman Collection

Import our Postman collection to test all endpoints:

[**Download Postman Collection →**](https://hasapay.com/postman-collection.json)

---

## Need Help?

- Check the [Authentication Guide](../documentation/authentication.md)
- Review [Error Codes](#http-status-codes)
- Contact support@hasapay.com
