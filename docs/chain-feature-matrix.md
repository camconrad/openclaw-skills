# Chain-feature matrix

Single source of truth for **which chains support which features**. Use this to route actions (e.g. only call Clanker when chain is in its set) or show friendly errors ("Token deployment via Clanker is only available on Base, Ethereum, Arbitrum, and Monad. Please switch chain.").

Assistant-supported chains are defined in [chains-config.md](chains-config.md). Not all features work on all chains.

---

## Per-feature chain support

| Feature / skill | Chain IDs | Chains (names) | Notes |
|----------------|-----------|----------------|-------|
| Arbitrary tx / generic Torque | All 12 | See [chains-config](chains-config.md) | Backend-dependent; use matrix below for feature-specific limits |
| **Clanker token deploy** | 1, 8453, 42161, 143 | Ethereum, Base, Arbitrum, Monad | Monad: static fees only. Not available on Abstract, Plasma, Linea, Hyper, Ink, BSC, Avalanche, Polygon. |
| **Polymarket** | 137 | Polygon | Polygon only (USDC.e) |
| **Endaoment donate** | 1, 10, 8453 | Ethereum, Optimism, Base | |
| **Veil** | 8453 | Base | Base only |
| **Botchan** | 8453 | Base | Base only |
| **Yoink** | 8453 | Base | Base only |
| **qrcoin** | 8453 | Base | Base only |
| **ENS primary name** | 1, 10, 42161, 8453 | Ethereum, Optimism, Arbitrum, Base | |

---

## How to use

- **Before suggesting token deploy (Clanker):** Check current chain is in {1, 8453, 42161, 143}. If not (e.g. Abstract), prompt user to switch to Base, Ethereum, Arbitrum, or Monad, or suggest another action.
- **Before Polymarket actions:** Check chain is Polygon (137). If not, prompt to switch to Polygon.
- **App/backend:** Before calling a protocol (Clanker, Endaoment, etc.), check chain is in that row’s set; if not, return a friendly "not available on this chain" message.

When you add or remove a chain for a protocol (e.g. Clanker adds Abstract), update this matrix first, then update the skill’s SKILL.md and references that reference it.
