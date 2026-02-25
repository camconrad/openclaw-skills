# Chains configuration (assistant supported chains)

Single source of truth for which chains the assistant can access in this skills repo. Replace the previous Bankr default (Base, Ethereum, Polygon only) with this set.

---

## 1. Chain IDs (use these constants everywhere)

| Chain     | Chain ID |
|-----------|----------|
| Ethereum  | 1        |
| BSC       | 56       |
| Polygon   | 137      |
| Monad     | 143      |
| Arbitrum  | 42161    |
| Avalanche | 43114    |
| Base      | 8453     |
| Plasma    | 9745     |
| Hyper     | 999      |
| Abstract  | 2741     |
| Linea     | 59144    |
| Ink       | 57073    |

---

## 2. Network names (for token lookups)

| Chain     | Network name |
|-----------|--------------|
| Ethereum  | ethereum     |
| BSC       | bsc          |
| Polygon   | polygon      |
| Monad     | monad        |
| Arbitrum  | arbitrum     |
| Avalanche | avalanche    |
| Base      | base         |
| Plasma    | plasma       |
| Hyper     | hyper        |
| Abstract  | abstract     |
| Linea     | linea        |
| Ink       | ink          |

---

## 3. RPC URLs and explorers (reference)

| Chain     | ID     | RPC URL (example)                   | Explorer URL                    |
|-----------|--------|-------------------------------------|----------------------------------|
| Ethereum  | 1      | https://eth.drpc.org                | https://etherscan.io             |
| Base      | 8453   | https://mainnet.base.org            | https://basescan.org             |
| Monad     | 143    | https://monad-mainnet.drpc.org      | https://monadvision.com          |
| Polygon   | 137    | https://polygon.drpc.org            | https://polygonscan.com          |
| Plasma    | 9745   | https://plasma.drpc.org             | https://explorer.plasma.io       |
| Linea     | 59144  | https://linea-rpc.publicnode.com    | https://lineascan.build          |
| Hyper     | 999    | https://rpc.countzero.xyz/evm       | https://explorer.hyper.xyz       |
| Abstract  | 2741   | https://api.mainnet.abs.xyz         | https://explorer.abstract.xyz    |
| Ink       | 57073  | https://ink.drpc.org                | https://explorer.inkonchain.com  |

Arbitrum (42161), Avalanche (43114), BSC (56): use your app’s or wagmi defaults.

---

## 4. Per-feature chain support

Not all features are available on all chains. For routing and user messaging (e.g. "Token deployment via Clanker is only available on Base, Ethereum, Arbitrum, and Monad"), see **[chain-feature-matrix.md](chain-feature-matrix.md)**.

For **token symbols, addresses per network, decimals, and lookups** (e.g. USDC, WETH, stablecoins), see **[token-config-reference.md](token-config-reference.md)**. Token addresses are keyed by the same network names as in the table above; the app uses `getNetworkFromChainId(chainId)` and `getTokenAddressForChain(symbol, chainId)`.

---

## 5. Summary

- **Assistant-supported chains:** Ethereum, BSC, Polygon, Monad, Arbitrum, Avalanche, Base, Plasma, Hyper, Abstract, Linea, Ink (12 EVM chains).
- **Chain IDs:** 1, 56, 137, 143, 999, 2741, 42161, 43114, 8453, 9745, 59144, 57073.
- **Per-feature limits:** Clanker (Base, Ethereum, Arbitrum, Monad only), Polymarket (Polygon only), Endaoment (Base, Ethereum, Optimism), Veil/Botchan/Yoink/qrcoin (Base only), ENS (Ethereum, Optimism, Arbitrum, Base). See [chain-feature-matrix.md](chain-feature-matrix.md).

Keep this file in sync with your app’s `lib/utils/chains.ts` (or equivalent). All Torque skill docs and references in this repo should use this list.
