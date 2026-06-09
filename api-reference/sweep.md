# Sweep API

Configure when and how HasaPay moves funds from child addresses back to the master wallet (auto-sweeping). Includes org defaults, per-chain overrides, per-address settings, history, and manual trigger.

**Authentication:** Dual-auth (JWT or HMAC)
**Base path:** `/api/v1/sweep`

---

## Sweep modes

HasaPay supports three behaviors, derived from two booleans (`auto_sweep_enabled`, `retain_child_balance`):

| Mode | `auto_sweep_enabled` | `retain_child_balance` | Behavior |
|---|---|---|---|
| `no_sweep` | `false` | `true` *(forced)* | Funds stay on child addresses. No automated movement. |
| `sweep_retain` | `true` | `true` | Sweep funds to master, **child ledger persists** (P2P model — child can still withdraw). |
| `sweep_clear` | `true` | `false` | Sweep funds to master, **child ledger zeroes** (payment-processor model — child is a one-shot receiver). |

> **Forbidden combination:** `auto_sweep_enabled=false` + `retain_child_balance=false` is rejected. With sweeping off, balances *must* stay on the child or they'd be unreachable. The server force-corrects to `retain=true` if you set `auto_sweep=false`.

### Sweep mechanism (per chain)

The mechanism HasaPay uses for a sweep is auto-detected per token at sweep time — you don't configure it. EVM stablecoins with EIP-2612 permit use a one-tx `permit + transferFrom`; non-permit ERC-20s use a multi-tx `fund_and_approve` flow with cached approvals; Tron TRC-20 uses a `fund_and_sweep` flow (master funds child with TRX, child broadcasts the TRC-20 transfer).

> **Note:** `sweep_model` was a config knob in earlier versions and was dropped in migration 035. If your client still sends it, it's ignored.

---

## Get org sweep config

```
GET /api/v1/sweep/config
```

Returns the org-level defaults plus any per-chain overrides.

### Response

```json
{
  "data": {
    "auto_sweep_enabled": true,
    "sweep_threshold_usd": 25,
    "sweep_min_native_reserve": "10000000000000000",
    "retain_child_balance": false,
    "effective_retain_child_balance": false,
    "mode": "sweep_clear",
    "chain_overrides": [
      {
        "id": "uuid",
        "organization_id": "uuid",
        "chain": "tron",
        "network": "mainnet",
        "auto_sweep_enabled": true,
        "sweep_threshold_usd": 50,
        "sweep_min_native_reserve": "1000000",
        "retain_child_balance": null,
        "created_at": "2026-06-08T10:00:00Z",
        "updated_at": "2026-06-08T10:00:00Z"
      }
    ]
  }
}
```

| Field | Meaning |
|---|---|
| `auto_sweep_enabled` | Org default — applied to chains that don't have an override |
| `sweep_threshold_usd` | Min USD value to trigger a sweep |
| `sweep_min_native_reserve` | Native amount to leave on child for gas (raw integer in the smallest unit — wei for EVM, sun for Tron) |
| `retain_child_balance` | Org default policy |
| `effective_retain_child_balance` | What the server actually uses (after the `auto_sweep=false` force-correction) |
| `mode` | Convenience label: `no_sweep`, `sweep_retain`, or `sweep_clear` |
| `chain_overrides[]` | Per-`(chain, network)` rows. `null` field = inherit org default |

---

## Update org sweep config

```
PUT /api/v1/sweep/config
```

All fields optional. Send only what you want to change.

### Request body

| Field | Type | Description |
|---|---|---|
| `auto_sweep_enabled` | bool | Master toggle |
| `sweep_threshold_usd` | float | Min USD to trigger (must be ≥ 0) |
| `sweep_min_native_reserve` | string | Raw native units to leave on child for gas |
| `retain_child_balance` | bool | `true` = `sweep_retain`, `false` = `sweep_clear` |

### Response

Returns the updated config plus the recomputed `mode`.

---

## Per-chain overrides

A row in `org_chain_configs` overrides the org default for one `(chain, network)`. `NULL` fields in the row mean "inherit from org default."

### Get one chain's override

```
GET /api/v1/sweep/config/:chain/:network
```

`404 not_found` when no override exists — caller should fall back to org defaults from `GET /sweep/config`.

### Upsert chain override

```
PUT /api/v1/sweep/config/:chain/:network
```

Idempotent. Same body as the org-level update (`auto_sweep_enabled`, `sweep_threshold_usd`, `sweep_min_native_reserve`, `retain_child_balance`). The OFF+OFF guard considers **effective** values — if your override leaves a field NULL, the org default fills in.

#### Example: set a chain-specific Tron threshold

```bash
curl -X PUT https://apitest.hasapay.com/api/v1/sweep/config/tron/mainnet \
  -H "Authorization: Bearer $JWT" \
  -H "Content-Type: application/json" \
  -d '{
    "auto_sweep_enabled": true,
    "sweep_threshold_usd": 50,
    "sweep_min_native_reserve": "1000000"
  }'
```

### Delete chain override (revert to inherit)

```
DELETE /api/v1/sweep/config/:chain/:network
```

Idempotent — no error if no row exists. After delete, the chain inherits org defaults again.

---

## Per-address sweep settings

### List addresses with their effective sweep settings

```
GET /api/v1/sweep/addresses?chain=&network=&limit=50&offset=0
```

### Response

```json
{
  "data": {
    "addresses": [
      {
        "id": "uuid",
        "address": "0x8ba1f109...",
        "chain": "ethereum",
        "network": "sepolia",
        "auto_sweep_enabled": true,
        "is_active": true
      }
    ],
    "total": 124,
    "limit": 50,
    "offset": 0
  }
}
```

### Update one address's sweep settings

```
PUT /api/v1/sweep/addresses/:addressId
```

#### Request body

| Field | Type | Description |
|---|---|---|
| `auto_sweep_enabled` | bool | Per-address override; `null` = inherit chain/org default |
| `sweep_threshold` | string | Per-address USD threshold; `null` = inherit |

---

## Manual sweep trigger

```
POST /api/v1/sweep/addresses/:addressId/trigger
```

Force a sweep of one address now, regardless of the threshold. Useful for ops use or for a UI "sweep now" button.

### Request body

| Field | Type | Required | Description |
|---|---|---|---|
| `token_symbol` | string | Yes | Which token to sweep (e.g. `USDC`) |

### Response

```json
{
  "data": {
    "sweep_id": "uuid",
    "status": "initiated"
  }
}
```

A successful trigger fires the same `sweep.initiated` / `sweep.completed` / `sweep.failed` webhook sequence as an automatic sweep.

---

## Sweep history

```
GET /api/v1/sweep/history?chain=&status=&limit=50&offset=0
```

### Query params

| Param | Description |
|---|---|
| `chain` / `network` | Filter |
| `status` | `pending`, `processing`, `completed`, `failed` |
| `from_date` / `to_date` | ISO 8601 |
| `limit` / `offset` | Pagination (default 50 / 0) |

### Response

```json
{
  "data": {
    "sweeps": [
      {
        "id": "uuid",
        "organization_id": "uuid",
        "source_type": "child_address",
        "source_id": "uuid",
        "destination_type": "master_wallet",
        "destination_id": "uuid",
        "chain": "ethereum",
        "network": "sepolia",
        "amount": "100000000",
        "token_symbol": "USDC",
        "token_decimals": 6,
        "status": "completed",
        "trigger_reason": "threshold",
        "created_at": "2026-06-09T10:00:00Z",
        "updated_at": "2026-06-09T10:05:00Z"
      }
    ],
    "total": 312,
    "limit": 50,
    "offset": 0
  }
}
```

`amount` is in raw token units — divide by `10^token_decimals` for display.

---

## Trigger reasons

| Reason | When |
|---|---|
| `threshold` | Auto — balance crossed `sweep_threshold_usd` |
| `manual` | Triggered via `POST /sweep/addresses/:id/trigger` |
| `scheduled` | Scheduled batch sweep (org config-driven) |
| `cleanup` | Leftover-funds sweep after a Tron TRC-20 sweep |

---

## Tron specifics — fund-and-sweep

TRC-20 sweeps work differently from EVM because child addresses can't pay energy until they have TRX:

1. Master estimates TRC-20 sweep cost (energy in SUN).
2. Master sends `max(0, needed − child_TRX_balance) × 1.3` TRX to the child.
3. Wait for confirmation.
4. Child broadcasts the TRC-20 `transfer(master, amount)` and pays its own energy from the funded TRX.
5. Wait for confirmation.
6. If `retain_child_balance=false` and leftover TRX > 1, sweep leftover TRX back to master.

Failure handling is documented in the source (`internal/service/gasless_orchestrator_tron.go`). The relevant ledger groups are `fund_for_sweep:{sweepID}`, `sweep:{sweepID}`, and `sweep_leftover:{sweepID}`.

> Native TRX sweeps don't need fund-and-sweep — the child pays bandwidth from the reserve we leave it during the previous sweep (`NativeReserve` on the Tron adapter = 1 TRX).

---

## Errors

| Code | Cause |
|---|---|
| `validation_error` | `sweep_threshold_usd` negative; or the forbidden OFF+OFF config |
| `not_found` | Chain override doesn't exist (GET only); address not on this org |
| `forbidden` | Address belongs to a different org |
| `internal_error` | Underlying repo/service failure — retryable |
