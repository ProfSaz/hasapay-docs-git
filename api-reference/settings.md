# Settings API

Manage organization settings.

**Authentication:** JWT

---

## Get Organization Settings

Get current organization settings.

```
GET /api/v1/settings/organization
```

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/settings/organization \
  -H "Authorization: Bearer {{TOKEN}}"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "organization": {
      "id": "org_456",
      "name": "Your Company Inc",
      "business_name": "Your Company Inc",
      "website": "https://yourcompany.com",
      "incorporation_country": "US",
      "industry": "fintech",
      "created_at": "2024-04-16T10:00:00Z",
      "updated_at": "2024-04-16T10:00:00Z"
    }
  }
}
```

---

## Update Organization Settings

Update organization details.

```
PUT /api/v1/settings/organization
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | Display name |
| `website` | string | No | Company website |

### Example Request

```bash
curl -X PUT https://apitest.hasapay.com/api/v1/settings/organization \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -d '{
    "name": "Updated Company Name",
    "website": "https://newwebsite.com"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "organization": {
      "id": "org_456",
      "name": "Updated Company Name",
      "website": "https://newwebsite.com",
      "updated_at": "2024-04-18T10:00:00Z"
    }
  }
}
```

---

## Errors

| Code | Description |
|------|-------------|
| `INVALID_URL` | Invalid website URL format |
| `PERMISSION_DENIED` | Only admins can update settings |
