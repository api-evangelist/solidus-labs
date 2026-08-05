# TokenSniffer — Webhooks Reference

Webhooks are an **Enterprise-only** feature that delivers real-time push notifications
to your server when token events occur. Instead of polling, your server receives a POST
request within seconds of the event.

Base URL: `https://tokensniffer.com/api/v3/webhooks`
Authentication: `?apikey=<key>` query parameter on all management requests.

---

## Event Types

| Event Type ID | Description | Status |
|---|---|---|
| `TOKEN.CREATED` | New ERC-20 contract deployed | Available |
| `TOKEN.MALICIOUS` | Token flagged as a scam | Available |
| `LIQUIDITY.CREATED` | New liquidity pool deployed | Available |
| `LIQUIDITY.REMOVED` | Sudden liquidity removal (rugpull) detected | Available |
| `TOKEN.BALANCE.CHANGE` | Holder balance change >5% | Coming soon |
| `TOKEN.SUPPLY.CHANGE` | Token supply change (mint/burn) | Coming soon |

---

## Creating a Webhook

```
POST https://tokensniffer.com/api/v3/webhooks?apikey=your_api_key
Content-Type: application/json
```

**Request body:**

```json
{
  "callback_url": "https://your-server.com/ts-webhook",
  "event_types": ["TOKEN.MALICIOUS", "LIQUIDITY.REMOVED"],
  "secret": "a-high-entropy-random-string",
  "filters": {
    "chain_id": 8453
  }
}
```

| Field | Required | Description |
|---|---|---|
| `callback_url` | ✅ | Your endpoint (HTTPS only). Must respond within 5 seconds with 200 or 204. |
| `event_types` | ✅ | Array of event type IDs to subscribe to. |
| `secret` | ✅ | Used for SHA-256 HMAC signing of notifications. Use a high-entropy random string. |
| `filters` | optional | Object to narrow which events are sent. Supports `chain_id: <int>` and `chain_ids: [<int>]`. |

**Response:**

```json
{
  "message": "OK",
  "subscription_id": "1fe6b82a-db29-4880-86ad-f79c57724c72",
  "callback_url": "https://your-server.com/ts-webhook",
  "event_types": ["TOKEN.MALICIOUS", "LIQUIDITY.REMOVED"],
  "created_at": "2025-11-26T20:47:38.161Z"
}
```

Save the `subscription_id` — you'll need it to update or delete the webhook.

---

## Managing Webhooks

### List all webhooks
```
GET /api/v3/webhooks?apikey=your_api_key&event_type=TOKEN.MALICIOUS&limit=100&offset=0
```

### Get a specific webhook
```
GET /api/v3/webhooks/{subscription_id}?apikey=your_api_key
```

### Update a webhook
```
PATCH /api/v3/webhooks/{subscription_id}?apikey=your_api_key
```
Body: the fields to change (same shape as create).

### Delete a webhook
```
DELETE /api/v3/webhooks/{subscription_id}?apikey=your_api_key
```

---

## Incoming Notification Format

When an event fires, TokenSniffer sends a POST request to your `callback_url`.
Events are **batched** and sent every 60 seconds, with up to 5,000 events per request.

**Request body:**

```json
{
  "total": 1,
  "events": [
    {
      "id": "224a9fe2-4313-49d0-b24b-0cdbaf200e9d",
      "type": "TOKEN.MALICIOUS",
      "timestamp": "2025-09-08T01:46:36.436Z",
      "data": {
        "chain_id": 8453,
        "token_address": "0x861d95981c53856245ccdfe0fb5f4848cd130894",
        "exploits": [
          {
            "id": 1002,
            "name": "Serial rug pull",
            "types": "rugpull"
          }
        ]
      }
    }
  ]
}
```

### Event-Specific `data` Fields

#### `TOKEN.CREATED`
| Field | Type | Description |
|---|---|---|
| `chain_id` | integer | Chain where the token was deployed |
| `token_address` | string | Token contract address |
| `name` | string | Token name |
| `symbol` | string | Token symbol |

#### `TOKEN.MALICIOUS`
| Field | Type | Description |
|---|---|---|
| `chain_id` | integer | Chain where the token is deployed |
| `token_address` | string | Token contract address |
| `exploits` | array | List of exploit objects with `id`, `name`, and `types` |

#### `LIQUIDITY.CREATED`
| Field | Type | Description |
|---|---|---|
| `chain_id` | integer | Chain where the pool was deployed |
| `pool_address` | string | Liquidity pool address |
| `token_address` | string | First token in the pair |
| `token_address` | string | Second token in the pair |

#### `LIQUIDITY.REMOVED`
| Field | Type | Description |
|---|---|---|
| `chain_id` | integer | Chain |
| `pool_address` | string | Pool where liquidity was removed |
| `token_address` | string | First token in the pair |
| `token_address` | string | Second token in the pair |

---

## Delivery Guarantees

- **Timeout:** Your endpoint must respond with HTTP 200 or 204 within **5 seconds** or the
  request is considered failed.
- **Retry logic:** On failure (non-2xx or timeout), the system retries up to **3 times**.
- **Idempotency:** Use the `id` field on each event to deduplicate — retries will send
  the same `id` values. Store processed IDs to avoid double-processing.
- **Batching:** Each POST may contain up to 5,000 events. Always iterate the `events[]`
  array, not just the first element.

---

## Verifying Requests

The `secret` you provided at creation is used to sign notifications with SHA-256 HMAC.
Verify the signature of incoming requests to ensure they originate from TokenSniffer
and not a third party.

---

## IP Allowlist

If your firewall restricts inbound traffic, allow these source IPs:

```
54.173.174.186
54.156.31.250
```

---

## Implementation Checklist

When building a webhook receiver:

1. Accept POST requests at your `callback_url`
2. Return 200 or 204 within 5 seconds — do heavy processing asynchronously
3. Verify the HMAC signature using your `secret`
4. Use `event.id` to deduplicate (idempotency)
5. Iterate `events[]` — each request may contain multiple events
6. Allowlist the TokenSniffer source IPs if needed
