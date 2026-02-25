# Token Trading Reference

Execute token trades and swaps across multiple blockchains.

## Supported Chains

See [docs/chains-config.md](../../docs/chains-config.md). Supported EVM chains: Ethereum (1), BSC (56), Polygon (137), Monad (143), Arbitrum (42161), Avalanche (43114), Base (8453), Plasma (9745), Hyper (999), Abstract (2741), Linea (59144), Ink (57073).

## Amount Formats

| Format | Example | Description |
|--------|---------|-------------|
| USD | `$50` | Dollar amount to spend |
| Percentage | `50%` | Percentage of your balance |
| Exact | `0.1 ETH` | Specific token amount |

## Prompt Examples

**Same-chain swaps:**
- "Swap 0.1 ETH for USDC on Base"
- "Buy $50 of BNKR on Base"
- "Sell 50% of my ETH holdings"
- "Purchase 100 USDC worth of PEPE"

**Cross-chain swaps:**
- "Bridge 0.5 ETH from Ethereum to Base"
- "Move 100 USDC from Polygon to Base"

**ETH/WETH conversion:**
- "Convert 0.1 ETH to WETH"
- "Unwrap 0.5 WETH to ETH"

## Chain Selection

- If no chain specified, Torque selects the most appropriate chain
- Base is preferred for most operations due to low fees
- Cross-chain routes are automatically optimized
- Include chain name in prompt to specify: "Buy ETH on Polygon"

## Slippage

- Default slippage tolerance is applied automatically
- For volatile tokens, Torque adjusts slippage as needed
- If slippage is exceeded, the transaction fails safely
- You can specify: "with 1% slippage"

## Common Issues

| Issue | Resolution |
|-------|------------|
| Insufficient balance | Reduce amount or add funds |
| Token not found | Check token symbol/address, specify chain |
| High slippage | Try smaller amounts or use limit orders |
| Network congestion | Wait and retry, or try L2 |
| Gas too high | Use Base/Polygon, or wait for lower gas |

## Best Practices

1. **Start small** - Test with small amounts first
2. **Specify chains** - For lesser-known tokens, always include chain
3. **Check slippage** - Be careful with low-liquidity tokens
4. **Monitor gas** - Ethereum mainnet can be expensive
5. **Use L2s** - Base and Polygon offer much lower fees
