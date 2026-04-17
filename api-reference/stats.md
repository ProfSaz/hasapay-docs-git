# Stats API

Get dashboard statistics and analytics.

**Authentication:** HMAC

---

## Get Organization Stats

Get overall statistics for your organization.

```
GET /api/v1/stats
```

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/stats \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "wallets": {
      "total": 5,
      "active": 5
    },
    "addresses": {
      "total": 150,
      "active": 145
    },
    "transactions": {
      "total": 1250,
      "deposits": 800,
      "withdrawals": 350,
      "sweeps": 100
    },
    "volume": {
      "total_usd": "125000.00",
      "deposits_usd": "100000.00",
      "withdrawals_usd": "25000.00"
    },
    "balances": {
      "total_usd": "75000.00"
    }
  }
}
```

---

## Get Volume History

Get transaction volume over time.

```
GET /api/v1/stats/volume
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `period` | string | `30d` | Time period: `7d`, `30d`, `90d`, `1y` |
| `chain` | string | - | Filter by chain |

### Example Request

```bash
curl -X GET "https://apitest.hasapay.com/api/v1/stats/volume?period=30d" \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "period": "30d",
    "data_points": [
      {
        "date": "2024-04-01",
        "deposits_usd": "5000.00",
        "withdrawals_usd": "1200.00",
        "total_usd": "6200.00"
      },
      {
        "date": "2024-04-02",
        "deposits_usd": "3500.00",
        "withdrawals_usd": "800.00",
        "total_usd": "4300.00"
      }
    ],
    "summary": {
      "total_deposits_usd": "100000.00",
      "total_withdrawals_usd": "25000.00",
      "total_volume_usd": "125000.00"
    }
  }
}
```

---

## Get Chain Breakdown

Get statistics broken down by blockchain.

```
GET /api/v1/stats/chains
```

### Example Request

```bash
curl -X GET https://apitest.hasapay.com/api/v1/stats/chains \
  -H "X-API-Key: {{API_KEY}}" \
  -H "X-Timestamp: {{TIMESTAMP}}" \
  -H "X-Request-ID: {{REQUEST_ID}}" \
  -H "X-Signature: {{SIGNATURE}}"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "chains": [
      {
        "chain": "ethereum",
        "network": "mainnet",
        "wallets": 2,
        "addresses": 50,
        "transactions": 500,
        "volume_usd": "75000.00",
        "balance_usd": "45000.00"
      },
      {
        "chain": "tron",
        "network": "mainnet",
        "wallets": 1,
        "addresses": 75,
        "transactions": 600,
        "volume_usd": "40000.00",
        "balance_usd": "25000.00"
      },
      {
        "chain": "polygon",
        "network": "mainnet",
        "wallets": 2,
        "addresses": 25,
        "transactions": 150,
        "volume_usd": "10000.00",
        "balance_usd": "5000.00"
      }
    ]
  }
}
```

---

## Errors

| Code | Description |
|------|-------------|
| `INVALID_PERIOD` | Invalid time period |
| `INVALID_CHAIN` | Unknown chain |
