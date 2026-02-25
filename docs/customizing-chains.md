# Customizing Supported Chains

**Single source of truth for assistant-supported chains:** [docs/chains-config.md](chains-config.md). That file lists the 12 EVM chains this fork supports (Ethereum, BSC, Polygon, Monad, Arbitrum, Avalanche, Base, Plasma, Hyper, Abstract, Linea, Ink). Update that file first; then keep the following in sync.

## 1. Torque skill (EVM)

**Docs (what the agent is told is supported):**

| File | What to change |
|------|----------------|
| `docs/chains-config.md` | **Edit this first.** Chain IDs, network names, RPC/explorer reference. |
| `torque/SKILL.md` | Description and **Supported Chains** table; link to `docs/chains-config.md`. |
| `torque/references/arbitrary-transaction.md` | **Supported Chains** table and `chainId` validation; link to chains-config. |
| `torque/references/safety.md` | Funding table; link to chains-config. |
| `torque/references/error-handling.md` | Balance recommendations; link to chains-config. |
| `torque/references/token-trading.md`, `portfolio.md`, `transfers.md`, `market-research.md`, `nft-operations.md`, `sign-submit-api.md` | Supported chains text; link to chains-config. |

**Note:** This fork defines assistant-supported chains in the skills repo (replacing the previous Bankr default of Base, Ethereum, Polygon only). The Torque backend/API must support these chains for actions to succeed.

---

## 2. ENS Primary Name

**Scripts (chain → chainId + RPC + registrar):**

| File | What to change |
|------|----------------|
| `ens-primary-name/scripts/set-primary.sh` | `case "$CHAIN" in` block: add/remove branches (e.g. `base) CHAIN_ID=8453; ...`, `arbitrum) CHAIN_ID=42161; ...`). Update the "Supported: base, arbitrum, ..." message. |
| `ens-primary-name/scripts/verify-primary.sh` | Same `case "$CHAIN" in` block — keep in sync with `set-primary.sh`. |

Add a new chain by adding a new `chainname)` block with the correct `CHAIN_ID`, `RPC_URL`, `REVERSE_REGISTRAR` (and `EXPLORER` if used).

---

## 3. ERC-8004 (agent registry)

**Scripts (mainnet vs testnet):**

| File | What to change |
|------|----------------|
| `erc-8004/scripts/register.sh` | `--testnet` branch: `CHAIN="sepolia"`, `CHAIN_ID=11155111`; default: `CHAIN="ethereum"`, `CHAIN_ID=1`. Change to your testnet/mainnet names and IDs. |
| `erc-8004/scripts/register-onchain.sh` | Same `--testnet` / default `CHAIN` and `CHAIN_ID`. |
| `erc-8004/scripts/register-http.sh` | Same. |
| `erc-8004/scripts/update-profile.sh` | Same. |
| `erc-8004/scripts/get-agent.sh` | `CHAIN_ID=1` or `11155111` — set to the chain your registry uses. |

---

## 4. Endaoment (donations)

**Scripts:**

| File | What to change |
|------|----------------|
| `endaoment/scripts/donate.sh` | Top of script: `CHAIN_ID=8453` (Base). Change to your default chain ID. Endaoment supports Base, Ethereum, Optimism; use the chain ID for the network you want. |
| `endaoment/scripts/search.sh` | `case "$CHAIN" in`: `ethereum|mainnet) CHAIN_IDX=0`, `optimism) CHAIN_IDX=1`, `base) CHAIN_IDX=2`. Indexes map to Endaoment’s deployment order; add/remove only if their API supports more chains. |

---

## 5. Botchan (onchain messaging)

**Docs + env:**

| File | What to change |
|------|----------------|
| `botchan/SKILL.md` | `BOTCHAN_CHAIN_ID=8453` (Base); "chain 8453" in examples; "default: 8453 for Base" in option table. For a different chain, set the chain ID and update the doc. |

Chain is controlled at runtime by the `BOTCHAN_CHAIN_ID` env var; the skill doc just needs to reflect your supported chain(s).

---

## 6. Clanker (token deployment)

**Docs:**

| File | What to change |
|------|----------------|
| `clanker/SKILL.md` | Supported chains table (Base 8453, Ethereum 1, Arbitrum 42161); examples with `chainId: 8453`. |
| `clanker/references/deployment.md` | `import { base } from 'viem/chains'` and `chain: base` — switch to your preferred chain(s) from viem. |

Actual support depends on Clanker SDK; align the docs with the chains your deployment target supports.

---

## 7. Veil

Veil Cash is Base-only. No chain customization unless the protocol adds more chains; then update `veil/SKILL.md` and any RPC/chain mentions.

---

## 8. Yoink, qrcoin

These skills are Base-specific (game/auction contracts). To support another chain you’d need contract addresses and to update the relevant SKILL.md and scripts.

---

## When a protocol only supports a subset of chains

Not all features work on all 12 assistant-supported chains. Use **[chain-feature-matrix.md](chain-feature-matrix.md)** as the single source of truth for "is feature X available on chain Y?"

**When a protocol only supports a subset of chains:**

1. **Update the chain-feature matrix first.** Edit [chain-feature-matrix.md](chain-feature-matrix.md) to add/remove the chain for that feature.
2. **Update the skill’s SKILL.md and references** to state the limited chains and link to the matrix (e.g. "Token deployment via Clanker is only available on Base, Ethereum, Arbitrum, and Monad; see [chain-feature-matrix](chain-feature-matrix.md).").
3. **App/backend:** If your app routes by chain (e.g. before calling Clanker), use the matrix to return a friendly "not available on this chain" message (e.g. "Token deployment via Clanker is only available on Base, Ethereum, Arbitrum, and Monad. Please switch chain.").

**Affected skills (per-feature chain limits):** Clanker (Base, Ethereum, Arbitrum, Monad only), Endaoment (Base, Ethereum, Optimism), Polymarket (Polygon only), Veil (Base only), Botchan (Base only), Yoink (Base only), qrcoin (Base only), ENS primary name (Ethereum, Optimism, Arbitrum, Base).

---

## Quick reference: assistant-supported chain IDs (this fork)

See [chains-config.md](chains-config.md) for the full list. Summary:

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

## Optional: single config for scripts

There is no shared `chains.json` or env file today; each skill’s scripts set `CHAIN_ID` (or equivalent) themselves. To centralize:

1. Add e.g. `config/chains.json` or a `.env` with `DEFAULT_CHAIN_ID=8453` and any chain list.
2. In each script that uses a chain, source that config (e.g. `source "$REPO_ROOT/config/chains.env"` or parse JSON) and use the variables instead of hardcoded IDs.
3. Keep `docs/customizing-chains.md` (this file) updated so others know where to add or remove chains.

Until then, use this doc as the checklist for customizing supported chains across the repo.
