# TokenSniffer — Smell Test Reference

The Smell Test is TokenSniffer's automated scoring system. Each test evaluates a specific
risk factor. Results appear in the `tests[]` array when `include_tests=true` is set.

## How to Read Results

**Critical note on result polarity:** Test descriptions are stated as the *desired
property*. A `result: false` means the token **failed** to satisfy that property and
the factor is a risk. A `result: true` means it passed.

Example: `testForMint` with description `"Source does not contain a mint function"`:
- `result: true` → no mint function found (good)
- `result: false` → mint function IS present (risk flag)

The `score` (0–100, where 0 = highest risk) summarizes all test results.

---

## Test List

### Contract Code Tests

| Test ID | Description | Notes |
|---|---|---|
| `testForMissingSource` | Verified contract source | Fails if source is NOT verified on a scan site. Unverified = bytecode-only analysis with lower confidence. |
| `testForProxy` | Source does not contain a proxy contract | Fails if a proxy/upgradeable pattern is detected. Logic can be swapped post-deploy. |
| `testForPausable` | Source does not contain a pausable contract | Fails if trading can be paused by an owner function. |
| `testForMint` | Source does not contain a mint function | Fails if a mint function exists. Supply can be inflated and dumped. |
| `testForRestoreOwnership` | Source does not contain a function to restore ownership | Fails if there is a hidden path to reclaim ownership after it's been renounced. |
| `testForMaxTransactionAmount` | Source does not contain a function to set maximum transaction amount | Fails if max tx/wallet amount can be set to near-zero, preventing sells. |
| `testForModifiableFee` | Source does not contain a function to modify the fee | Fails if fees can be changed. Risk of setting to 100%. |
| `testForBlacklist` | Source does not contain a function to blacklist holders | Fails if specific addresses can be blocked from transferring or selling. |

### Ownership Tests

| Test ID | Description | Notes |
|---|---|---|
| `testForOwnershipNotRenounced` | Ownership renounced or source does not contain an owner contract | Fails if ownership is still held. Owner can call privileged functions. |
| `testForAuthorization` | Creator not authorized for special permission | Fails if the deployer has special permissions (e.g., bypass transfer restrictions). |

### Token Balance Tests

| Test ID | Description | Additional Fields | Notes |
|---|---|---|---|
| `testForTokensLockedOrBurned` | Tokens locked/burned | `value`, `value_pct` | Checks that a meaningful portion of supply is locked or burned. Low value_pct = concentrated supply. |
| `testForHighCreatorTokenBalance` | Creator wallet contains less than 5% of token supply | `value`, `value_pct` | Fails if deployer holds ≥5% of supply. High creator balance = dump risk. |
| `testForHighOwnerTokenBalance` | Owner wallet contains less than 5% of token supply | `value`, `value_pct` | Fails if current owner holds ≥5% of supply. |
| `testForHighWalletTokenBalance` | All other wallets contain less than 5% of token supply | `data` | Fails if any single non-creator/owner wallet holds ≥5% of supply. |
| `testForBurnedBalanceExceedsSupply` | Burned amount exceeds total token supply | — | Sanity check. Fails if burned > total supply (data anomaly). |
| `testForCombinedWalletsExceedSupply` | All wallets combined contain less than 100% of token supply | — | Sanity check for supply accounting. |
| `testForImpossibleWalletTokenBalance` | All wallets contain less than 100% of token supply | — | Sanity check — any single wallet holding 100%+ is anomalous. |

### Liquidity Tests

| Test ID | Description | Additional Fields | Notes |
|---|---|---|---|
| `testForInadequateLiquidity` | Adequate current liquidity | `value`, `value_pct`, `currency` | Fails if current liquidity is below a meaningful threshold. `value` is the raw amount; `currency` is `"ETH"`, `"BNB"`, etc. |
| `testForInadequateInitialLiquidity` | Adequate initial liquidity | `value_pct` | Fails if the initial liquidity added to the pool was very low. |
| `testForInadeqateLiquidityLockedOrBurned` | At least 95% of liquidity locked/burned | `value`, `value_pct` | The key LP safety test. Fails if `(lock_balance + burn_balance) / total_supply < 0.95`. Low lock % = rugpull risk. |
| `testForHighCreatorLPBalance` | Creator wallet contains less than 5% of liquidity | `value`, `value_pct` | Fails if deployer holds ≥5% of LP tokens. |
| `testForHighOwnerLPBalance` | Owner wallet contains less than 5% of liquidity | `value`, `value_pct` | Fails if owner holds ≥5% of LP tokens. |

### Swap Simulation Tests

| Test ID | Description | Additional Fields | Notes |
|---|---|---|---|
| `testForUnableToSell` | Token is sellable | — | Fails if simulated sell swap fails — the definitive honeypot test. Only available on ETH, Base, BSC, Blast. |
| `testForHighBuyFee` | Buy fee is less than 5% | `value_pct` | Fails if buy fee ≥5%. `value_pct` is the fee fraction (0.05 = 5%). |
| `testForHighSellFee` | Sell fee is less than 5% | `value_pct` | Fails if sell fee ≥5%. |
| `testForExtremeFee` | Buy/sell fee is less than 30% | — | Fails if either fee ≥30%. This is a hard stop — extreme fees are never legitimate. |

---

## Interpreting a Failed Test for a User

When explaining a failed test, map `result: false` to the risk it represents:

| Test ID | Failed → means |
|---|---|
| `testForUnableToSell` | **Honeypot confirmed** — token cannot be sold |
| `testForExtremeFee` | Buy or sell fee ≥30% — effectively a trap |
| `testForMint` | Creator can print unlimited supply and dump |
| `testForBlacklist` | Creator can prevent any wallet from selling |
| `testForPausable` | Creator can freeze all trading |
| `testForProxy` | Contract logic can be changed to anything after deployment |
| `testForInadeqateLiquidityLockedOrBurned` | Creator can drain the liquidity pool at any time |
| `testForHighCreatorTokenBalance` | Creator holds a large stake — likely to dump |
| `testForMissingSource` | Cannot verify what the contract actually does |
| `testForOwnershipNotRenounced` | Owner still holds privileged access |
