# TokenSniffer API — Get Token Response Fields

Full description of every field returned by `GET /api/v2/tokens/{chain_id}/{address}`.
Fields marked **(metrics)** require `include_metrics=true`. Fields marked **(tests)** require `include_tests=true`.

---

## Top-Level Fields (always present)

| Field | Type | Description |
|---|---|---|
| `message` | string | `"OK"` on success, or an error message |
| `status` | string | `"ready"` when data is complete; `"pending"` when analysis is still running. Retry after 5s if pending, or use `block_until_ready=true`. |
| `chain_id` | integer | The provided chain identifier |
| `address` | string | The provided contract address |
| `name` | string | Token name string |
| `symbol` | string | Token symbol string |
| `total_supply` | number | Token total supply |
| `decimals` | integer | Token decimals |
| `created_at` | integer | Milliseconds since epoch when contract was deployed |
| `refreshed_at` | integer \| null | Milliseconds since epoch when token data was last updated; null if not yet updated |
| `deployer_addr` | string | Address that deployed the contract |
| `is_flagged` | boolean | `true` = confirmed scam, either by automated detection or moderator review. Score is forced to 0 when flagged. |
| `flagged_at` | integer \| null | Milliseconds since epoch when contract was flagged as a scam; null if not flagged |
| `is_suspect` | boolean | `true` = suspicious code found but not confirmed as a scam |
| `is_pending` | boolean | `true` = flagged for manual review |
| `is_rugpull` | boolean | `true` = at least one liquidity pool experienced a rugpull event |
| `exploits` | string[] | Exploit type strings from automated scam detection (e.g. `"honeypot"`, `"hidden mint"`). See `exploit-types.md` for definitions. |

---

## Contract Fields (always present)

All under `contract.*`.

| Field | Type | Description |
|---|---|---|
| `contract.is_source_verified` | boolean | `true` = contract source code is verified on a scan site (Etherscan, etc.). `false` = only bytecode analysis was possible — increases uncertainty. |
| `contract.has_mint` | boolean | A mint function was detected. Owner can create new tokens, enabling a dump. |
| `contract.has_fee_modifier` | boolean | A fee-modifying function was detected. Fees can be cranked to 100%, trapping sellers. |
| `contract.has_max_transaction_amount` | boolean | A function to set maximum transaction/wallet amounts exists. Can be set to near-zero, preventing sells. |
| `contract.has_blocklist` | boolean | An allowlist/blocklist function exists. Specific addresses can be blocked from selling. |
| `contract.has_proxy` | boolean | Proxy/upgradeable pattern detected. Contract logic can be swapped for malicious code post-deploy. |
| `contract.has_pausable` | boolean | A function to pause trading was detected. Can freeze the market at will. |
| `contract.has_nonstandard_ledger_variable_name` | boolean | The `balanceOf` function uses a nonstandard internal variable name — a technique used to hide manipulable balance storage. |
| `contract.has_nonstandard_transfer_function_signature` | boolean | `transfer`/`transferFrom` have nonstandard arguments — a technique to intercept or redirect transfers. |

---

## Metrics Fields (require `include_metrics=true`)

### Score and Risk

| Field | Type | Description |
|---|---|---|
| `score` | integer | 0–100 rug pull risk estimate. **0 = most risky, 100 = least risky.** Always 0 when `is_flagged=true`. |
| `risk_level` | string | `"low"` (score ≥85), `"medium"` (score ≥60), `"high"` (score <60) |

### Permissions

Under `permissions.*`:

| Field | Type | Description |
|---|---|---|
| `permissions.owner_address` | string \| null | Current value of the owner variable, or null if not present |
| `permissions.is_ownership_renounced` | boolean | `true` = ownership transferred to zero address (owner can no longer call privileged functions) |

### Swap Simulation

Under `swap_simulation.*`. Only available on Ethereum, Base, BNB Chain, and Blast, and only when a liquidity pool exists. Will be `null` if it can't be determined.

| Field | Type | Description |
|---|---|---|
| `swap_simulation.is_sellable` | boolean \| null | `false` = token is a honeypot — cannot be sold in a simulated swap |
| `swap_simulation.buy_fee` | integer \| null | Buy fee percentage from simulated swap. >5% is suspicious; 100% = all funds go to fee wallet |
| `swap_simulation.sell_fee` | integer \| null | Sell fee percentage from simulated swap. >5% is suspicious; 100% = confirmed trap |

### Balances

Under `balances.*`:

| Field | Type | Description |
|---|---|---|
| `balances.burn_balance` | number | Tokens held by null/burn addresses (dead supply) |
| `balances.lock_balance` | number | Tokens locked in a locker contract |
| `balances.deployer_balance` | number | Tokens held by the deployer address. High values = rug risk. |
| `balances.owner_balance` | number | Tokens held by the current owner (0 if ownership renounced) |
| `balances.top_holders` | array | Top 20 token holders |
| `balances.top_holders[].address` | string | Holder address |
| `balances.top_holders[].balance` | number | Number of tokens held |
| `balances.top_holders[].is_contract` | boolean | `true` = this holder is a contract, not an EOA |

### Pools

`pools` is an array of liquidity pool objects. Each pool:

| Field | Type | Description |
|---|---|---|
| `pools[].address` | string | LP contract address |
| `pools[].name` | string | DEX name (e.g. `"Uniswap v2"`, `"PancakeSwap"`) |
| `pools[].base_symbol` | string | Symbol of the paired token (`"ETH"`, `"USDC"`, etc.) |
| `pools[].base_address` | string | Address of the paired token |
| `pools[].total_supply` | number | Total supply of LP tokens |
| `pools[].decimals` | integer | LP token decimals |
| `pools[].base_reserve` | number | Current liquidity in terms of the base token (e.g., ETH amount) |
| `pools[].initial_base_reserve` | number | Initial liquidity added soon after pool creation |
| `pools[].owner_balance` | number | LP tokens held by the owner address |
| `pools[].deployer_balance` | number | LP tokens held by the deployer address |
| `pools[].burn_balance` | number | LP tokens burned (held by null/burn addresses) |
| `pools[].lock_balance` | number | LP tokens locked in locker contracts |
| `pools[].top_holders` | array | Top 20 LP token holders (same shape as `balances.top_holders`) |
| `pools[].locks` | array | Array of lock descriptors |
| `pools[].locks[].address` | string | Address of the LP locker contract |
| `pools[].locks[].name` | string | Locker product name (e.g. `"Unicrypt"`, `"TrustSwap"`) |
| `pools[].locks[].url` | string | Locker product homepage URL |
| `pools[].locks[].balance` | number | LP tokens held by this locker |
| `pools[].locks[].start_time` | integer \| null | Unix timestamp of lock start; null if indeterminate |
| `pools[].locks[].end_time` | integer \| null | Unix timestamp of unlock; null if indeterminate |

**Liquidity safety heuristic:** `(burn_balance + lock_balance) / total_supply` should be ≥0.95 (95%) to pass the liquidity lock test.

---

## Smell Test Fields (require `include_tests=true`)

The `score` and `risk_level` fields are also present (same values as with `include_metrics`).

`tests` is an array of test descriptors:

| Field | Type | Description |
|---|---|---|
| `tests[].id` | string | Unique test identifier (e.g. `"testForMint"`) |
| `tests[].description` | string | Human-readable description of what the test checks. **Note:** descriptions are stated as the *desired* property. A `result: false` means the token *failed* to satisfy that property (e.g. `"Source does not contain a mint function"` with `result: false` means it DOES have a mint function). |
| `tests[].result` | boolean | `false` = test failed (this is a risk factor). `true` = test passed. |
| `tests[].value` | number | (some tests only) Absolute value measured |
| `tests[].value_pct` | number | (some tests only) Fractional value (0.0–1.0) |
| `tests[].data` | any | (some tests only) Structured extra data |
| `tests[].currency` | string | (some tests only) Currency symbol for the value (e.g. `"ETH"`) |

See `smell-tests.md` for the complete list of test IDs and what each one checks.

---

## Similar Contracts (require `include_similar=true`)

`similar` is an array:

| Field | Type | Description |
|---|---|---|
| `similar[].chain_id` | string | Chain ID of the similar contract |
| `similar[].address` | string | Address of the similar contract |
| `similar[].score` | integer | Similarity score (0–100) |
