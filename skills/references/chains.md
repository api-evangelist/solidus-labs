# TokenSniffer — Supported Chains, DEXes, and Lockers

## Supported Chains

| Chain ID | Name | Alias(es) |
|---|---|---|
| 1 | Ethereum | `eth`, `ethereum` |
| 10 | Optimism | `opt`, `optimism` |
| 25 | Cronos | `cronos` |
| 56 | BNB Smart Chain (BSC) | `bsc` |
| 100 | Gnosis (xDAI) | `gnosis`, `xdai` |
| 101 | Solana | `solana` |
| 137 | Polygon | `poly`, `polygon` |
| 250 | Fantom | `ftm`, `fantom` |
| 321 | KuCoin Community Chain (KCC) | `kcc` |
| 8453 | Base | `base` |
| 42161 | Arbitrum | `arb`, `arbitrum` |
| 42262 | Oasis | `oasis` |
| 43114 | Avalanche | `avax`, `avalanche` |
| 81457 | Blast | `blast` |
| 1666600000 | Harmony | `harmony` |

Use `GET /api/v2/chains` (no auth required) to retrieve the live list.

**Note:** The numeric `chain_id` integer is what the API requires for path parameters.
Aliases are accepted in some query parameters but integers are always safe to use.

---

## Supported DEXes (for pool/liquidity data)

| Chain | Supported DEXes |
|---|---|
| Solana | Pump.fun, PumpSwap, Raydium LPv4/CPMM/CLMM, Orca WP, Meteora DYN/DLMM |
| Ethereum | Uniswap v2/v3, SushiSwap v2, ShibaSwap |
| Base | Uniswap v2/v3, PancakeSwap v2/v3, SushiSwap v2, BaseSwap, RocketSwap, LeetSwap |
| BNB Smart Chain (BSC) | Uniswap v2/v3, PancakeSwap v1/v2/v3, ApeSwap, PadSwap |
| Polygon | Uniswap v2/v3, QuickSwap, ApeSwap |
| Arbitrum | Uniswap v2/v3, SushiSwap v2 |
| Optimism | Uniswap v2/v3 |
| Fantom | SpiritSwap, SpookySwap |
| Avalanche | Uniswap v2/v3, Pangolin, Trader Joe |
| Blast | Thruster v2 |
| KuCoin Community Chain | KoffeeSwap |
| Cronos | Crodex, CronaSwap |
| Oasis | YuzuSwap |
| Gnosis | *Coming soon* |
| Harmony | *Coming soon* |

---

## Supported Liquidity Lockers (for lock detection in pool data)

LP lock data appears in `pools[].locks[]` in the Get Token response.

| Chain | Supported Lockers |
|---|---|
| Ethereum | Team Finance, UNCX, Unilocker, PinkLock, OnlyMoons, GemPad |
| Base | Team Finance, UNCX, PinkLock, BaseSwap Locker, OnlyMoons, GemPad |
| BNB Smart Chain (BSC) | Team Finance, UNCX, PinkLock, Mudra, DeepLock, Unilocker, CryptEx, OnlyMoons, GemPad |
| Arbitrum | UNCX, OnlyMoons, GemPad |
| Polygon | Team Finance, UNCX, Unilocker, OnlyMoons, GemPad |
| Fantom | Team Finance |
| Avalanche | UNCX, OnlyMoons |
| Cronos | Hibiki |
| Optimism | *Coming soon* |
| Blast | *Coming soon* |
| Gnosis | *Coming soon* |
| Harmony | *Coming soon* |
| KCC | *Coming soon* |
| Oasis | *Coming soon* |

---

## Chains with Swap Simulation

Swap simulation (`swap_simulation.*` fields) is only available on:
- Ethereum (chain 1)
- Base (chain 8453)
- BNB Smart Chain (chain 56)
- Blast (chain 81457)

On other chains, `swap_simulation.is_sellable`, `buy_fee`, and `sell_fee` will be `null`.

---

## Chains with Full Smell Test Support

Metrics and Smell Test results (`include_metrics=true`, `include_tests=true`) are only
available for:

Ethereum, BSC, Base, Blast, Polygon, Fantom, Arbitrum, Avalanche, Cronos, Oasis.

These flags are **not supported** on Solana, Optimism, Gnosis, Harmony, or KCC.
