---
name: tokensniffer-api
description: >
  Deep knowledge of the TokenSniffer REST API for token safety analysis, scam detection,
  and DeFi integration. Use this skill whenever someone is working with the TokenSniffer
  API — checking if a token is a scam, integrating token safety scores into an app,
  understanding Smell Test results, writing code to call any TokenSniffer endpoint,
  interpreting risk scores or exploit flags, setting up webhooks for scam alerts, or
  building any feature that consumes TokenSniffer data. Also use it when a user asks
  about on-chain token metrics, rugpull detection, honeypot detection, or liquidity
  analysis via TokenSniffer — even if they don't say "TokenSniffer" explicitly.
---

# TokenSniffer API

TokenSniffer is the leading EVM token security platform. It analyzes smart contract source
code and bytecode against a database of 10,000+ scam patterns, flags 50–75% of all new
tokens as scams, and provides scored risk assessments (Smell Test) for on-chain metrics
like liquidity, holder concentration, and swap simulation.

All requests go to `https://tokensniffer.com`. The API has two versions:
- `v2` — tokens, addresses, chains, pairs
- `v3` — webhooks

## Authentication

Every request requires an API key passed as the `apikey` query parameter:

```http
GET /api/v2/tokens/1/0xabc...?apikey=your_api_key_here
```

**Never expose the key client-side.** All requests must be server-side — the query parameter containing the key must never appear in frontend code or browser requests.

## Plan Tiers

| Feature | Pro ($99/mo) | Enterprise |
|---|---|---|
| Tokens/day | 500 | 5,000+ |
| Get Token | ✅ | ✅ |
| Get Address | ✅ | ✅ |
| List Latest/Malicious Tokens | ❌ | ✅ |
| List Latest/Malicious Pairs | ❌ | ✅ |
| List Malicious Addresses | ❌ | ✅ |
| Webhooks | ❌ | ✅ |

Rate limit: **5 requests/second**. At most **5 concurrent** Get Token requests can be in
`pending` status simultaneously. Exceeding either limit returns HTTP 429.

Only unique token requests count toward the daily limit — repeated lookups of the same
token are free. Failed requests (non-200) are also not counted.

---

## Core Endpoint: Get Token

```
GET https://tokensniffer.com/api/v2/tokens/{chain_id}/{address}
```

This is the primary endpoint. It returns scam status, contract analysis, and optionally
on-chain metrics and individual Smell Test results.

### Query Parameters

| Parameter | Type | Default | Purpose |
|---|---|---|---|
| `include_metrics` | bool | false | Add score, risk_level, swap simulation, holder balances, pools |
| `include_tests` | bool | false | Add individual Smell Test pass/fail results |
| `include_similar` | bool | false | Add similar contract matches (for pattern detection) |
| `block_until_ready` | bool | false | Long-poll until status="ready" instead of returning "pending" |

**When to use each flag:**
- `include_metrics=true` — any time you need the risk score, sell fee, liquidity, or holder
  concentration. Adds 0.1–5s latency (slower when refresh is needed every 5–30 min).
- `include_tests=true` — when you need to show or evaluate individual test pass/fail
  results (e.g., "why is the score low?").
- `include_similar=true` — when building scam pattern detection or checking if a contract
  is a copy of a known scam.
- `block_until_ready=true` — for newly deployed tokens or when `include_metrics/tests` is
  set and you can't handle a `pending` response. Increases latency but guarantees a full
  result.

### Status Handling

The response includes a `status` field:
- `"ready"` — full data available, use it.
- `"pending"` — analysis is still running (new token or refresh in progress). **Retry after
  5 seconds** until `status="ready"`, or use `block_until_ready=true` to let the API wait.

New tokens are indexed within 30 seconds of deployment. A **404** for a brand-new token
means it hasn't been indexed yet — retry after 1–2 minutes.

### Reading the Result

The most important fields for a safety decision:

```
is_flagged     → true = confirmed scam (score is forced to 0)
exploits[]     → list of exploit types detected (honeypot, hidden mint, etc.)
score          → 0-100, where 0 = maximum risk, 100 = cleanest (requires include_metrics)
risk_level     → "high" (<60), "medium" (60+), "low" (85+)             (requires include_metrics)
is_rugpull     → a liquidity pool was drained
is_suspect     → suspicious code found but not confirmed scam
```

Contract-level red flags (always present, no extra flag needed):

```
contract.has_mint              → can create new tokens (dilution/dump risk)
contract.has_fee_modifier      → fees can be changed (could go to 100%)
contract.has_blocklist         → can block specific addresses from selling
contract.has_proxy             → upgradeable — logic can be swapped out
contract.has_pausable          → trading can be paused
contract.has_max_transaction_amount → can cap trade size (manipulation)
contract.is_source_verified    → false = bytecode-only analysis, higher uncertainty
```

When `include_metrics=true`, also check:

```
swap_simulation.is_sellable    → false = honeypot (can't sell)
swap_simulation.buy_fee        → integer percentage, >5% is suspicious
swap_simulation.sell_fee       → >5% is suspicious, 100% = confirmed trap
permissions.is_ownership_renounced → false = owner can still call privileged functions
balances.deployer_balance      → high = rug risk
pools[].lock_balance           → how much LP is locked (higher = safer)
```

---

## Other Endpoints

For full parameter lists and response field details, see `references/endpoints.md`.

### Get Address (Pro+)
```
GET https://tokensniffer.com/api/v2/addresses/{address}
```
Returns scam contracts deployed by or linked to an address. Use this to check a deployer's
reputation before trusting a new token they launched.

### List Supported Chains (no auth)
```
GET https://tokensniffer.com/api/v2/chains
```
Returns all chain IDs, names, and aliases. Use this to resolve chain identifiers. See
`references/chains.md` for the full table.

### Get Usage Statistics
```
GET https://tokensniffer.com/api/v2/usage
```
Returns `{ limit, used }` — daily request cap and how many have been consumed so far.

### Enterprise List Endpoints

All return `{ message, total, result[] }` with pagination via `limit` and `offset`.
Optional `chain_id`, `start_time`, `end_time` filters on all of them.

| Endpoint | Path |
|---|---|
| List Latest Tokens | `GET /api/v2/tokens/latest` |
| List Malicious Tokens | `GET /api/v2/tokens/scams` |
| List Corrected Tokens | `GET /api/v2/tokens/corrections` |
| List Latest Pairs | `GET /api/v2/pairs/latest` |
| List Malicious Pairs | `GET /api/v2/pairs/scams` |
| List Tokens w/ Malicious Pairs | `GET /api/v2/pairs/scams/tokens` |
| List Malicious Addresses | `GET /api/v2/addresses/scams` |

---

## Common Workflows

### Quick Safety Check

The fastest path: just call Get Token with no extra flags. Check `is_flagged`, `exploits`,
and the `contract.*` booleans. This is sufficient to reject obvious scams.

```python
import httpx

def quick_check(chain_id: int, address: str, api_key: str) -> dict:
    resp = httpx.get(
        f"https://tokensniffer.com/api/v2/tokens/{chain_id}/{address}",
        params={"apikey": api_key},
        timeout=10,
    )
    resp.raise_for_status()
    data = resp.json()

    if data.get("status") == "pending":
        # retry after 5s or re-call with block_until_ready=true
        pass

    return {
        "is_safe": not data.get("is_flagged") and not data.get("is_suspect"),
        "exploits": data.get("exploits", []),
        "contract_flags": data.get("contract", {}),
    }
```

### Deep Token Analysis

Use `include_metrics=true&include_tests=true` to get the full picture — score, risk level,
individual test results, and on-chain data. Expect 0.1–5s response time.

Key things to surface to a user from a deep analysis:
1. `score` and `risk_level` as the headline
2. Any `exploits` — always a hard stop
3. `swap_simulation.is_sellable` — honeypot check
4. `swap_simulation.sell_fee` — anything >5% is worth flagging
5. Liquidity lock: `pools[].lock_balance / pools[].total_supply` — aim for >95%
6. `balances.deployer_balance` as a fraction of `total_supply` — aim for <5%
7. Failed `tests[]` — explain each one to the user in plain language

See `references/smell-tests.md` for the full test list and what each failure means.
See `references/response-fields.md` for every field description.

### New Token Monitoring (Enterprise)

Poll `GET /api/v2/tokens/latest` every minute. For each new token, call Get Token with
`include_metrics=true`. Flag immediately if `is_flagged=true`. Build a queue — don't
exceed 5 concurrent pending requests.

### Deployer Reputation Check

Before trusting a new token, look up its `deployer_address` with Get Address:
```
GET /api/v2/addresses/{deployer_address}
```
A non-zero `malicious_contracts_count` is a strong red flag — serial scammers reuse
wallets.

### Webhook-Driven Alerting (Enterprise)

Set up a webhook to receive real-time scam alerts without polling:
```
POST https://tokensniffer.com/api/v3/webhooks
```
See `references/webhooks.md` for the full setup, event types, and how to verify incoming
requests with the shared secret.

---

## Error Handling

| HTTP Status | Meaning | Action |
|---|---|---|
| 200 | OK | Check `status` field — may still be "pending" |
| 404 | Not found | New token not yet indexed — retry in 1–2 min |
| 401 | Unauthorized | API key missing or expired |
| 403 | Permission denied | Endpoint requires higher plan tier |
| 429 | Rate limit | Back off — reduce req/sec or concurrent pending |
| 500 | Server error | Retry with exponential backoff, contact support |

---

## Reference Files

Read these when you need to go deeper:

- `references/endpoints.md` — full parameter lists and response schemas for every endpoint
- `references/response-fields.md` — every field in the Get Token response, described
- `references/exploit-types.md` — what each exploit string (honeypot, hidden mint, etc.) means mechanically
- `references/smell-tests.md` — all 26 Smell Test IDs, descriptions, and extra fields
- `references/chains.md` — all chain IDs, aliases, supported DEXes and liquidity lockers
- `references/webhooks.md` — webhook management, event types, notification format, IP allowlist
