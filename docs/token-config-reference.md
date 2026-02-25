# Token configuration reference

Single source of truth for **token symbols, addresses per network, decimals, logos, and CoinGecko IDs** lives in your app’s `lib/utils/token-config.ts`. Chain IDs and network names are in `lib/utils/chains.ts`. This doc summarizes the full supported set and conventions so skills and assistant behavior stay aligned with the app.

---

## 1. Network names (address keys)

Token addresses are keyed by **network name**, not chain ID. Use `getNetworkFromChainId(chainId)` from `lib/utils/chains.ts`. Network names must match [chains-config](chains-config.md):

| Network   | Chain ID |
|-----------|----------|
| ethereum  | 1        |
| arbitrum  | 42161    |
| base      | 8453     |
| bsc       | 56       |
| avalanche | 43114    |
| monad     | 143      |
| polygon   | 137      |
| plasma    | 9745     |
| linea     | 59144    |
| hyper     | 999      |
| abstract  | 2741     |
| ink       | 57073    |

Resolve a token on a chain: `getNetworkFromChainId(chainId)` → then `tokenConfig.address[network]`. Do not key by chain ID.

---

## 2. TokenConfig shape

Every entry in `TOKEN_CONFIGS` (in the app) uses this shape:

```ts
export interface TokenConfig {
  symbol: string
  name: string
  decimals: number
  address: Record<string, string>  // network -> address
  logo?: string
  coingeckoId?: string              // CoinGecko API ID for price fetching
}
```

`TOKEN_CONFIGS` is keyed by **symbol**. All lookups should use the helpers in §3.

---

## 3. Helper functions (app: `lib/utils/token-config.ts`)

| Function | Purpose |
|----------|---------|
| `getTokenConfig(symbol)` | Full config or null. Case-insensitive symbol fallback. |
| `getTokenDecimals(symbol, network?)` | Decimals; default 18 if unknown. |
| `getTokenAddress(symbol, network)` | Contract address for symbol on network, or null. |
| `getTokenAddressForChain(symbol, chainId)` | Address for symbol on chain (uses `getNetworkFromChainId`); handles native token; ETH on chainId 1 returns zero address. |
| `isTokenSupported(symbol, network)` | Whether symbol has an address on the network. |
| `getSupportedTokens(network)` | Symbols that have an address on the network. |
| `getSupportedNetworks(symbol)` | Networks for which the token has an address. |
| `getTokenLogo(symbol, chainId?)` | Logo path or null; resolves native token by chainId; aToken prefix (e.g. AUSDC) uses underlying logo. |
| `getTokenImageClassName(symbol, baseClassName?)` | Optional dark-mode inversion (e.g. AAPL); returns class string. |
| `getCoinGeckoId(symbol)` | CoinGecko ID for price APIs; uses TOKEN_CONFIGS then fallback map (ETH, WETH, BTC, WBTC, aTokens, etc.). |
| `isStockToken(symbol)` | True if symbol is in `STOCK_TOKENS` (excluded from default token lists). |
| `isNativeToken(symbol, chainId)` | True if symbol is the native token for that chain. |

Re-exported from chains: `getNativeTokenSymbol`, `getNativeTokenAddress`, `getNativeTokenForChain`. Native tokens are not in `TOKEN_CONFIGS`; they are in `lib/utils/chains.ts` (`NATIVE_TOKENS`).

---

## 4. Decimals conventions

- **Stablecoins (USD):** usually `6` (USDC, USDT, USDbC, USDC.e, syrupUSDC/USDT, EURC, PYUSD, USDH 18, USD1 18, USDS/sUSDS 18, GHO/crvUSD 18, USDe/sUSDe 18).
- **BTC:** `8` (BTC, cbBTC, WBTC, KBTC, UBTC, WBTCe); **tBTC:** `18`.
- **ETH and wrappers:** `18` (WETH, stETH, wstETH, cbETH, weETH, rETH, ezETH, etc.).
- **Chain/ecosystem and DeFi:** `18`.
- **Gold / RWA:** XAUT/XAUt0 `6`, PAXG `18`; trillions `18`.
- **Stock tokens:** `18`.

---

## 5. Supported tokens (by category)

Canonical list lives in app’s `TOKEN_CONFIGS`; below is the supported symbol set and categories for skills/docs.

### Stablecoins (6 decimals unless noted)

USDC, USDbC, USDC.e, syrupUSDC, syrupUSDT, EURC, USDT, USDT0, USDtb, mUSD, PYUSD; USDH (18), USD1 (18), USDS (18), sUSDS (18), GHO (18), stkGHO (18), crvUSD (18), USDe (18), sUSDe (18).

### Bitcoin

BTC (8, no address key); cbBTC (8), tBTC (18), WBTC (8), KBTC (8), UBTC (8), WBTC.e (8).

### Ethereum derivatives (18 decimals)

cbETH, stETH, wstETH, weETH, rETH, ezETH, wstUSR, WETH.

### Chain / ecosystem

LINEA, UXPL, kHYPE, WHYPE, wstHYPE, stHYPE, vkHYPE, PENGU, STG, POL, AVAX, BNB, MNT, WMON, WSOL.

### DeFi (18 decimals)

AAVE, stkAAVE, MORPHO, LINK, BAL, ARB, VIRTUAL, ZORA, AERO, ZRO, PENDLE, CRV, cbXRP (6).

### Commodities / RWA

XAUT (6), XAUt0 (6), trillions (18), PAXG (18).

### Other

ONDO, ENA, sENA, SKY, HEMI, LDO, USDG, RLUSD, cUSD, stcUSD, WZEC (18), GMONAD, CHOG.

### Placeholder (commented in app)

TORQ, stTORQ — not deployed; addresses are placeholders. Re-enable in `TOKEN_CONFIGS` when live.

### Stock tokens (excluded from default lists)

In `TOKEN_CONFIGS` and in `STOCK_TOKENS`; use `isStockToken(symbol)` to exclude from swap/transfer selectors.

- **Technology:** AAPL, META, GOOGL, AMZ, NVDA, ORCL, SNAP, PLTR  
- **Automotive:** TSLA, RIVN  
- **Energy:** XOM, CVX  
- **Telecom:** T, TMUS, VZ  
- **Retail/Consumer:** WMT, HD, NKE, SBUX  
- **Entertainment/Media:** NFLX, GME  
- **Other/Platforms:** ABNB, UBER, DASH, CAT  
- **ETFs:** QQQ  

Currencies (display/cash subset) are in `lib/utils/currencies.ts` (`SUPPORTED_CURRENCIES`, `DEFAULT_CURRENCIES`, `CURRENCY_SYMBOLS`); codes should align with symbols in `TOKEN_CONFIGS` where they represent the same asset.

---

## 6. Stock tokens and list exclusion

`STOCK_TOKENS` is a `Set<string>` of symbols that are in `TOKEN_CONFIGS` but **excluded from default token lists** (e.g. swap/transfer). Use `isStockToken(symbol)` before including a token in a default list. Adding a new tokenized stock: add to `TOKEN_CONFIGS` and to `STOCK_TOKENS`.

---

## 7. Integration with chains and skills

- **Chains:** `getNetworkFromChainId(chainId)` (chains.ts) maps chain ID → network key for `tokenConfig.address[network]` and `getTokenAddressForChain`.
- **Skills:** When a skill or assistant flow involves a token (e.g. “swap USDC”, “approve USDC for Endaoment”, “Clanker paired token WETH”), use the same symbols and network names as in the app so addresses and decimals stay in sync.
- **Per-feature chains:** Use [chain-feature-matrix](chain-feature-matrix.md) for which chains support which features; use `isTokenSupported(symbol, network)` when needed.
- **Prices:** Price API uses `getCoinGeckoId(symbol)`; fallback map in token-config covers ETH, WETH, BTC, WBTC, native/ecosystem tokens, and aTokens (e.g. AUSDC, AWETH).

---

## 8. Summary

| Concept | Location (app) | Purpose |
|---------|----------------|---------|
| Token configs | `TOKEN_CONFIGS`, `TokenConfig` (token-config.ts) | Single source of truth for symbol, name, decimals, addresses, logo, coingeckoId |
| Network keys | `getNetworkFromChainId(chainId)` (chains.ts) | Keys for `address` in TokenConfig; same as [chains-config](chains-config.md) |
| Lookup by symbol + chain | `getTokenAddressForChain(symbol, chainId)` | Contract address; native and ETH placeholder handled |
| Support checks | `isTokenSupported(symbol, network)` | Whether a token exists on a network |
| Logos | `getTokenLogo(symbol, chainId?)`, `getTokenImageClassName(symbol, baseClassName?)` | Logo path; optional dark-mode class (e.g. AAPL) |
| Prices | `getCoinGeckoId(symbol)` | CoinGecko ID; fallback map for symbols not in TOKEN_CONFIGS |
| Stock tokens | `STOCK_TOKENS`, `isStockToken(symbol)` | Exclude from default token lists |
| Native tokens | `getNativeTokenForChain(chainId)` (chains.ts), `isNativeToken(symbol, chainId)` | Not in TOKEN_CONFIGS; resolved by chain |
| Currencies | `SUPPORTED_CURRENCIES`, `CURRENCY_SYMBOLS` (currencies.ts) | Display and default currency subset |

Keep this reference in sync with `lib/utils/token-config.ts` and `lib/utils/chains.ts`. When adding a new token or network in the app, update the app first; then update this doc and any skill docs that reference specific tokens or networks.
