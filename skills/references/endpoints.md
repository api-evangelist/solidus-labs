# TokenSniffer API — Endpoint Reference

Base URL: `https://tokensniffer.com`
Authentication: `?apikey=<key>` query parameter on every request (except List Supported Chains).

---

## Tokens

### Get Token
**Pro+** — `GET /api/v2/tokens/{chain_id}/{address}`

Returns token info, scam status, contract analysis, and optionally on-chain metrics and Smell Test results.

**Path parameters:**

| Name | Type | Required | Description |
|---|---|---|---|
| `chain_id` | integer | ✅ | Numeric chain identifier (e.g., 1 for Ethereum, 56 for BSC) |
| `address` | string | ✅ | Token contract address (checksummed or lowercase hex) |

**Query parameters:**

| Name | Type | Default | Description |
|---|---|---|---|
| `include_metrics` | bool | false | Include score, risk_level, permissions, swap_simulation, balances, pools |
| `include_tests` | bool | false | Include Smell Test individual results |
| `include_similar` | bool | false | Include similar contract matches |
| `block_until_ready` | bool | false | Long-poll until status="ready" |

**Response (always present):**

```json
{
  "message": "OK",
  "status": "ready",
  "chain_id": 56,
  "address": "0x...",
  "name": "ChildDoge",
  "symbol": "ChildDoge",
  "total_supply": 123,
  "decimals": 9,
  "created_at": 1624455637000,
  "refreshed_at": 1624455700000,
  "deployer_addr": "0x...",
  "is_flagged": true,
  "flagged_at": 1624455700000,
  "is_suspect": false,
  "is_pending": false,
  "is_rugpull": false,
  "exploits": ["honeypot"],
  "contract": {
    "is_source_verified": true,
    "has_mint": true,
    "has_fee_modifier": false,
    "has_max_transaction_amount": false,
    "has_blocklist": false,
    "has_proxy": false,
    "has_pausable": true,
    "has_nonstandard_ledger_variable_name": false,
    "has_nonstandard_transfer_function_signature": false
  }
}
```

**With `include_metrics=true`**, adds:

```json
{
  "score": 0,
  "risk_level": "high",
  "permissions": {
    "owner_address": "0x...",
    "is_ownership_renounced": false
  },
  "swap_simulation": {
    "is_sellable": true,
    "buy_fee": 1,
    "sell_fee": 0
  },
  "balances": {
    "burn_balance": 1234,
    "lock_balance": 1234,
    "deployer_balance": 1234,
    "owner_balance": 123,
    "top_holders": [
      { "address": "0x...", "balance": 71767430324769.23, "is_contract": false }
    ]
  },
  "pools": [
    {
      "address": "0x...",
      "name": "Uniswap v2",
      "base_address": "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2",
      "base_symbol": "ETH",
      "total_supply": 9472911695158690000000,
      "decimals": 18,
      "base_reserve": 5.004,
      "initial_base_reserve": 2.5,
      "owner_balance": 0,
      "deployer_balance": 7401466896810723000,
      "burn_balance": 219973065873940400000,
      "lock_balance": 9245537162387938000000,
      "locks": [
        {
          "address": "0x...",
          "name": "TrustSwap",
          "url": "https://trustswap.com",
          "balance": 9245537162387938000000,
          "start_time": 1673638451,
          "end_time": 1673638451
        }
      ],
      "top_holders": [
        { "address": "0x...", "balance": 71767430324769.23 }
      ]
    }
  ]
}
```

**With `include_tests=true`**, adds:

```json
{
  "score": 0,
  "risk_level": "high",
  "tests": [
    {
      "id": "testForInadeqateLiquidityLockedOrBurned",
      "description": "At least 95% of liquidity locked/burned",
      "result": false,
      "value": 9465510228261878000000,
      "value_pct": 0.9992
    }
  ]
}
```

**Response times:**
- Default (info + scam status): 0.1–0.4s
- With metrics/tests (cached): 0.1–0.4s
- With metrics/tests (refresh needed): 2–5s

---

### List Latest Tokens
**Enterprise** — `GET /api/v2/tokens/latest`

Get tokens created in the last 24 hours, updated in real-time.

**Query parameters:**

| Name | Type | Default | Description |
|---|---|---|---|
| `chain_id` | integer | — | Filter by chain |
| `limit` | integer | 100 | Max results (up to 5000) |
| `offset` | integer | 0 | Results to skip |
| `start_time` | string | — | ISO timestamp lower bound |
| `end_time` | string | — | ISO timestamp upper bound |

**Response:**

```json
{
  "message": "OK",
  "total": 5000,
  "result": [
    {
      "chain_id": "56",
      "address": "0x...",
      "deployer_addr": "0x...",
      "name": "The Yellow Emperor",
      "symbol": "Huangdi",
      "created_at": "2025-04-04T21:10:06.000Z",
      "exploits": []
    }
  ]
}
```

---

### List Malicious Tokens
**Enterprise** — `GET /api/v2/tokens/scams`

Get scam tokens flagged in the last 24 hours, updated in real-time.

**Query parameters:** Same as List Latest Tokens, plus:

| Name | Type | Description |
|---|---|---|
| `deployer_address` | string | Filter by deployer address |

**Response:**

```json
{
  "message": "OK",
  "total": 1,
  "result": [
    {
      "chain_id": 1,
      "address": "0x...",
      "name": "Frog Santa",
      "symbol": "FGS",
      "deployer_address": "0x...",
      "created_at": "2022-12-15T04:05:15.719Z",
      "flagged_at": "2022-12-15T05:00:00.000Z",
      "is_source_verified": true,
      "exploits": [
        {
          "id": 35,
          "name": "Hidden mint functionality #5",
          "types": ["fake ownership renounce", "hidden mint"]
        }
      ]
    }
  ]
}
```

---

### List Corrected Tokens
**Enterprise** — `GET /api/v2/tokens/corrections`

Tokens that were un-flagged in the last 24 hours (false positive corrections; <1% false-positive rate).

**Query parameters:** Same as List Latest Tokens.

**Response:**

```json
{
  "message": "OK",
  "total": 36,
  "result": [
    {
      "chain_id": "1",
      "address": "0x...",
      "deployer_addr": "0x...",
      "name": "SHIBA INU CHAIN",
      "symbol": "SINU",
      "created_at": "2023-03-05T18:22:35.000Z",
      "unflagged_at": "2025-04-04T20:02:09.144Z",
      "is_source_verified": true
    }
  ]
}
```

---

## Pairs

### List Latest Pairs
**Enterprise** — `GET /api/v2/pairs/latest`

Get trading pairs created in the last 24 hours.

**Query parameters:** Same as List Latest Tokens.

**Response:**

```json
{
  "message": "OK",
  "total": 5000,
  "result": [
    {
      "chain_id": "56",
      "factory_address": "0x...",
      "pair_address": "0x...",
      "created_at": "2025-04-04T21:10:06.000Z",
      "is_rugpull": false,
      "tokens": [
        { "address": "0x...", "name": "The Yellow Emperor", "symbol": "Huangdi" },
        { "address": "0x...", "name": "Wrapped BNB", "symbol": "WBNB" }
      ],
      "rugpulls": []
    }
  ]
}
```

---

### List Malicious Pairs
**Enterprise** — `GET /api/v2/pairs/scams`

Get pairs that experienced a rugpull in the last 24 hours.

**Query parameters:** Same as List Latest Tokens.

**Response:** Same shape as List Latest Pairs, with `is_rugpull: true` and populated `rugpulls[]`:

```json
"rugpulls": [
  {
    "category": "swap",
    "timestamp": "2025-04-04T21:06:06.000Z",
    "tx_hash": "0x..."
  }
]
```

---

### List Tokens with Malicious Pairs
**Enterprise** — `GET /api/v2/pairs/scams/tokens`

Get the non-quote tokens from rugpulled pairs (excludes ETH, BNB, etc.).

**Query parameters:** Same as List Latest Tokens.

**Response:**

```json
{
  "message": "OK",
  "total": 1357,
  "result": [
    {
      "address": "0x...",
      "name": "AIBRAIN",
      "symbol": "AIBRAIN",
      "chain_id": "56",
      "rugpulls": [
        {
          "category": "swap",
          "timestamp": "2025-04-04T21:06:06.000Z",
          "tx_hash": "0x..."
        }
      ]
    }
  ]
}
```

---

## Addresses

### Get Address
**Pro+** — `GET /api/v2/addresses/{address}`

Get scam tokens deployed by or associated with an address. Useful for checking deployer reputation.

**Path parameters:**

| Name | Type | Required | Description |
|---|---|---|---|
| `address` | string | ✅ | Contract address or EOA |

**Response:**

```json
{
  "message": "OK",
  "malicious_contracts_count": 1,
  "categories": [
    {
      "name": "scam contract",
      "types": ["blacklist", "hidden ownership", "honeypot"],
      "count": 1
    }
  ],
  "contracts": [
    {
      "chainId": 1,
      "address": "0x...",
      "types": ["honeypot"]
    }
  ]
}
```

---

### List Malicious Addresses
**Enterprise** — `GET /api/v2/addresses/scams`

Get addresses that deployed a known scam token in the last 24 hours.

**Query parameters:**

| Name | Type | Description |
|---|---|---|
| `chain_id` | string | Filter by chain |
| `limit` | integer | Max results (default 100) |
| `offset` | integer | Results to skip (default 0) |
| `start_time` | string | ISO timestamp lower bound |
| `end_time` | string | ISO timestamp upper bound |
| `deployer_address` | string | Filter to a specific deployer |

**Response:**

```json
{
  "message": "OK",
  "total": 123,
  "result": [
    {
      "deployer_address": "0x...",
      "flagged_contracts_count": 1,
      "reasons": ["malicious contract"]
    }
  ]
}
```

---

## Chains

### List Supported Chains
**No auth required** — `GET /api/v2/chains`

**Response:**

```json
{
  "total": 15,
  "result": [
    { "id": 1, "name": "Ethereum", "alias": "eth" },
    { "id": 56, "name": "BNB Smart Chain", "alias": "bsc" }
  ]
}
```

---

## Usage Statistics

`GET /api/v2/usage` (or similar path — check with your API key provider)

**Response:**

```json
{
  "limit": 500,
  "used": 42
}
```

---

## Webhooks

All webhook endpoints use `https://tokensniffer.com/api/v3/webhooks`.
See `webhooks.md` for the full reference including event types, payload format, and delivery details.

### Create Webhook
`POST /api/v3/webhooks`

**Request body:**

```json
{
  "callback_url": "https://your-server.com/webhook",
  "event_types": ["TOKEN.MALICIOUS", "LIQUIDITY.REMOVED"],
  "secret": "your-random-secret-string",
  "filters": { "chain_id": 8453 }
}
```

**Response:**

```json
{
  "message": "OK",
  "subscription_id": "1fe6b82a-db29-4880-86ad-f79c57724c72",
  "callback_url": "https://your-server.com/webhook",
  "event_types": ["TOKEN.MALICIOUS", "LIQUIDITY.REMOVED"],
  "created_at": "2025-11-26T20:47:38.161Z"
}
```

### List Webhooks
`GET /api/v3/webhooks`

Query params: `event_type`, `limit`, `offset`.

### Get Webhook
`GET /api/v3/webhooks/{subscription_id}`

### Update Webhook
`PATCH /api/v3/webhooks/{subscription_id}`

### Delete Webhook
`DELETE /api/v3/webhooks/{subscription_id}`
