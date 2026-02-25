---
name: torque
description: AI-powered crypto trading agent and LLM gateway via natural language. Use when the user wants to trade crypto, check portfolio balances, view token prices, transfer crypto, manage NFTs, use leverage, bet on Polymarket, deploy tokens, set up automated trading, sign and submit raw transactions, or access LLM models through the Torque LLM gateway funded by your Torque wallet. Supports Ethereum, Arbitrum, Base, BSC, Avalanche, Monad, Polygon, Plasma, Linea, Hyper, Abstract, and Ink (see [chains-config](../docs/chains-config.md)).
metadata:
  {
    "clawdbot":
      {
        "emoji": "📺",
        "homepage": "https://torque.fi",
        "requires": { "bins": ["torque"] },
      },
  }
---

# Torque

Execute crypto trading and DeFi operations using natural language. Two integration options:

1. **Torque CLI** (recommended) — Install `@torque/cli` for a batteries-included terminal experience
2. **REST API** — Call `https://api.torque.fi` directly from any language or tool

Both use the same API key and the same async job workflow under the hood.

## Getting an API Key

Before using either option, you need a Torque API key. Two ways to get one:

**Option A: Headless email login (recommended for agents)**

Two-step flow — send OTP, then verify and complete setup. See "First-Time Setup" below for the full guided flow with user preference prompts.

```bash
# Step 1 — send OTP to email
torque login email user@example.com

# Step 2 — verify OTP and generate API key (options based on user preferences)
torque login email user@example.com --code 123456 --accept-terms --key-name "My Agent" --read-write
```

This creates a wallet, accepts terms, and generates an API key — no browser needed. Before running step 2, ask the user whether they need read-only or read-write access, LLM gateway, and their preferred key name.

**Option B: Torque Assistant**

1. Visit [app.torque.fi/assistant](https://app.torque.fi/assistant)
2. **Sign up / Sign in** — Enter your email and the one-time passcode (OTP) sent to it
3. **Generate an API key** — Create a key with **Agent API** access enabled (the key starts with `bk_...`)

Both options automatically provision **EVM wallets** for all supported chains (see [chains-config](../docs/chains-config.md)) — no manual wallet setup needed.

## Option 1: Torque CLI (Recommended)

### Install

```bash
bun install -g @torque/cli
```

Or with npm:

```bash
npm install -g @torque/cli
```

### First-Time Setup

#### Headless email login (recommended for agents)

When the user asks to log in with an email, walk them through this flow:

**Step 1 — Send verification code**

```bash
torque login email <user-email>
```

**Step 2 — Ask the user for the OTP code** they received via email.

**Step 3 — Before completing login, ask the user about their preferences:**

1. **Accept Terms of Service** — Present the [Terms of Service](https://torque.fi/terms) link and confirm the user agrees. Required for new users — do not pass `--accept-terms` unless the user has explicitly confirmed.
2. **Read-only or read-write API key?**
   - **Read-only** (default) — portfolio, balances, prices, research only
   - **Read-write** (`--read-write`) — enables swaps, transfers, orders, token launches, leverage, Polymarket bets
3. **Enable LLM gateway access?** (`--llm`) — multi-model API at `llm.torque.fi` (currently limited to beta testers). Skip if user doesn't need it.
4. **Key name?** (`--key-name`) — a display name for the API key (e.g. "My Agent", "Trading Bot")

**Step 4 — Construct and run the step 2 command** with the user's choices:

```bash
# Example with all options
torque login email <user-email> --code <otp> --accept-terms --key-name "My Agent" --read-write --llm

# Example read-only, no LLM
torque login email <user-email> --code <otp> --accept-terms --key-name "Research Bot"
```

#### Login options reference

| Option | Description |
|--------|-------------|
| `--code <otp>` | OTP code received via email (step 2) |
| `--accept-terms` | Accept [Terms of Service](https://torque.fi/terms) without prompting (required for new users) |
| `--key-name <name>` | Display name for the API key (e.g. "My Agent"). Prompted if omitted |
| `--read-write` | Enable write operations: swaps, transfers, orders, token launches, leverage, Polymarket bets. **Without this flag, the key is read-only** (portfolio, balances, prices, research only) |
| `--llm` | Enable [LLM gateway](https://docs.torque.fi/llm-gateway/overview) access (multi-model API at `llm.torque.fi`). Currently limited to beta testers |

Any option not provided on the command line will be prompted interactively by the CLI, so you can mix headless and interactive as needed.

#### Login with existing API key

If the user already has an API key:

```bash
torque login --api-key bk_YOUR_KEY_HERE
```

If they need to create one at the Torque Terminal:
1. Run `torque login --url` — prints the assistant URL
2. Present the URL to the user, ask them to generate a `bk_...` key
3. Run `torque login --api-key bk_THE_KEY`

#### Separate LLM Gateway Key (Optional)

If your LLM gateway key differs from your API key, pass `--llm-key` during login or run `torque config set llmKey YOUR_LLM_KEY` afterward. When not set, the API key is used for both. See [references/llm-gateway.md](references/llm-gateway.md) for full details.

#### Verify Setup

```bash
torque whoami
torque prompt "What is my balance?"
```

## Option 2: REST API (Direct)

No CLI installation required — call the API directly with `curl`, `fetch`, or any HTTP client.

### Authentication

All requests require an `X-API-Key` header:

```bash
curl -X POST "https://api.torque.fi/agent/prompt" \
  -H "X-API-Key: bk_YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is my ETH balance?"}'
```

### Quick Example: Submit → Poll → Complete

```bash
# 1. Submit a prompt — returns a job ID
JOB=$(curl -s -X POST "https://api.torque.fi/agent/prompt" \
  -H "X-API-Key: $TORQUE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is my ETH balance?"}')
JOB_ID=$(echo "$JOB" | jq -r '.jobId')

# 2. Poll until terminal status
while true; do
  RESULT=$(curl -s "https://api.torque.fi/agent/job/$JOB_ID" \
    -H "X-API-Key: $TORQUE_API_KEY")
  STATUS=$(echo "$RESULT" | jq -r '.status')
  [ "$STATUS" = "completed" ] || [ "$STATUS" = "failed" ] || [ "$STATUS" = "cancelled" ] && break
  sleep 2
done

# 3. Read the response
echo "$RESULT" | jq -r '.response'
```

### Conversation Threads

Every prompt response includes a `threadId`. Pass it back to continue the conversation:

```bash
# Start — the response includes a threadId
curl -X POST "https://api.torque.fi/agent/prompt" \
  -H "X-API-Key: $TORQUE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is the price of ETH?"}'
# → {"jobId": "job_abc", "threadId": "thr_XYZ", ...}

# Continue — pass threadId to maintain context
curl -X POST "https://api.torque.fi/agent/prompt" \
  -H "X-API-Key: $TORQUE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "And what about SOL?", "threadId": "thr_XYZ"}'
```

Omit `threadId` to start a new conversation. CLI equivalent: `torque prompt --continue` (reuses last thread) or `torque prompt --thread <id>`.

### API Endpoints Summary

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/agent/prompt` | POST | Submit a prompt (async, returns job ID) |
| `/agent/job/{jobId}` | GET | Check job status and results |
| `/agent/job/{jobId}/cancel` | POST | Cancel a running job |
| `/agent/sign` | POST | Sign messages/transactions (sync) |
| `/agent/submit` | POST | Submit raw transactions (sync) |

For full API details (request/response schemas, job states, rich data, polling strategy), see:

**Reference**: [references/api-workflow.md](references/api-workflow.md) | [references/sign-submit-api.md](references/sign-submit-api.md)

## CLI Command Reference

### Core Commands

| Command | Description |
|---------|-------------|
| `torque login` | Authenticate with the Torque API (interactive menu) |
| `torque login email <address>` | Send OTP to email (headless step 1) |
| `torque login email <address> --code <otp> [options]` | Verify OTP and complete setup (headless step 2) |
| `torque login --api-key <key>` | Login with an existing API key directly |
| `torque login --api-key <key> --llm-key <key>` | Login with separate LLM gateway key |
| `torque login --url` | Print Torque Terminal URL for API key generation |
| `torque logout` | Clear stored credentials |
| `torque whoami` | Show current authentication info |
| `torque prompt <text>` | Send a prompt to the Torque AI agent |
| `torque prompt --continue <text>` | Continue the most recent conversation thread |
| `torque prompt --thread <id> <text>` | Continue a specific conversation thread |
| `torque status <jobId>` | Check the status of a running job |
| `torque cancel <jobId>` | Cancel a running job |
| `torque skills` | Show all Torque AI agent skills with examples |

### Configuration Commands

| Command | Description |
|---------|-------------|
| `torque config get [key]` | Get config value(s) |
| `torque config set <key> <value>` | Set a config value |
| `torque --config <path> <command>` | Use a custom config file path |

Valid config keys: `apiKey`, `apiUrl`, `llmKey`, `llmUrl`

Default config location: `~/.torque/config.json`. Override with `--config` or `TORQUE_CONFIG` env var.

### Environment Variables

| Variable | Description |
|----------|-------------|
| `TORQUE_API_KEY` | API key (overrides stored key) |
| `TORQUE_API_URL` | API URL (default: `https://api.torque.fi`) |
| `TORQUE_LLM_KEY` | LLM gateway key (falls back to `TORQUE_API_KEY` if not set) |
| `TORQUE_LLM_URL` | LLM gateway URL (default: `https://llm.torque.fi`) |

Environment variables override config file values. Config file values override defaults.

### LLM Gateway Commands

| Command | Description |
|---------|-------------|
| `torque llm models` | List available LLM models |
| `torque llm setup openclaw [--install]` | Generate or install OpenClaw config |
| `torque llm setup opencode [--install]` | Generate or install OpenCode config |
| `torque llm setup claude` | Show Claude Code environment setup |
| `torque llm setup cursor` | Show Cursor IDE setup instructions |
| `torque llm claude [args...]` | Launch Claude Code via the Torque LLM Gateway |

## Core Usage

### Simple Query

For straightforward requests that complete quickly:

```bash
torque prompt "What is my ETH balance?"
torque prompt "What's the price of Bitcoin?"
```

The CLI handles the full submit-poll-complete workflow automatically. You can also use the shorthand — any unrecognized command is treated as a prompt:

```bash
torque What is the price of ETH?
```

### Interactive Prompt

For prompts containing `$` or special characters that the shell would expand:

```bash
# Interactive mode — no shell expansion issues
torque prompt
# Then type: Buy $50 of ETH on Base

# Or pipe input
echo 'Buy $50 of ETH on Base' | torque prompt
```

### Conversation Threads

Continue a multi-turn conversation with the agent:

```bash
# First prompt — starts a new thread automatically
torque prompt "What is the price of ETH?"
# → Thread: thr_ABC123

# Continue the conversation (agent remembers the ETH context)
torque prompt --continue "And what about BTC?"
torque prompt -c "Compare them"

# Resume any thread by ID
torque prompt --thread thr_ABC123 "Show me ETH chart"
```

Thread IDs are automatically saved to config after each prompt. The `--continue` / `-c` flag reuses the last thread.

### Manual Job Control

For advanced use or long-running operations:

```bash
# Submit and get job ID
torque prompt "Buy $100 of ETH"
# → Job submitted: job_abc123

# Check status of a specific job
torque status job_abc123

# Cancel if needed
torque cancel job_abc123
```

## LLM Gateway

The [Torque LLM Gateway](https://docs.torque.fi/llm-gateway/overview) is a unified API for Claude, Gemini, GPT, and other models — multi-provider access, cost tracking, automatic failover, and SDK compatibility through a single endpoint.

**Base URL:** `https://llm.torque.fi`

Uses your `llmKey` if configured, otherwise falls back to your API key.

### Quick Commands

```bash
torque llm models                           # List available models
torque llm credits                          # Check credit balance
torque llm setup openclaw --install         # Install Torque provider into OpenClaw
torque llm setup opencode --install         # Install Torque provider into OpenCode
torque llm setup claude                     # Print Claude Code env vars
torque llm setup cursor                     # Cursor setup instructions
torque llm claude                           # Launch Claude Code through gateway
torque llm claude --model claude-opus-4.6   # Launch with specific model
```

### Direct SDK Usage

The gateway works with standard OpenAI and Anthropic SDKs — just override the base URL:

```bash
# OpenAI-compatible
curl -X POST "https://llm.torque.fi/v1/chat/completions" \
  -H "Authorization: Bearer $TORQUE_LLM_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "claude-sonnet-4.5", "messages": [{"role": "user", "content": "Hello"}]}'
```

For full model list, provider config JSON shape, SDK examples (Python, TypeScript), all setup commands, and troubleshooting, see:

**Reference**: [references/llm-gateway.md](references/llm-gateway.md)

## Capabilities Overview

### Trading Operations

- **Token Swaps**: Buy/sell/swap tokens across chains
- **Cross-Chain**: Bridge tokens between chains
- **Limit Orders**: Execute at target prices
- **Stop Loss**: Automatic sell protection
- **DCA**: Dollar-cost averaging strategies
- **TWAP**: Time-weighted average pricing

**Reference**: [references/token-trading.md](references/token-trading.md)

### Portfolio Management

- Check balances across all chains
- View USD valuations
- Track holdings by token or chain
- Real-time price updates
- Multi-chain aggregation

**Reference**: [references/portfolio.md](references/portfolio.md)

### Market Research

- Token prices and market data
- Technical analysis (RSI, MACD, etc.)
- Social sentiment analysis
- Price charts
- Trending tokens
- Token comparisons

**Reference**: [references/market-research.md](references/market-research.md)

### Transfers

- Send to addresses, ENS, or social handles
- Multi-chain support
- Flexible amount formats
- Social handle resolution (Twitter, Farcaster, Telegram)

**Reference**: [references/transfers.md](references/transfers.md)

### NFT Operations

- Browse and search collections
- View floor prices and listings
- Purchase NFTs via OpenSea
- View your NFT portfolio
- Transfer NFTs
- Mint from supported platforms

**Reference**: [references/nft-operations.md](references/nft-operations.md)

### Polymarket Betting

Polymarket is on **Polygon only** (see [chain-feature-matrix](../docs/chain-feature-matrix.md)).

- Search prediction markets
- Check odds
- Place bets on outcomes
- View positions
- Redeem winnings

**Reference**: [references/polymarket.md](references/polymarket.md)

### Leverage Trading

- Long/short positions (up to 50x crypto, 100x forex/commodities)
- Crypto, forex, and commodities
- Stop loss and take profit
- Position management via Avantis on Base

**Reference**: [references/leverage-trading.md](references/leverage-trading.md)

### Token Deployment

- **EVM (Clanker):** Deploy ERC20 tokens via Clanker **only on Base, Ethereum, Arbitrum, and Monad** (Monad: static fees only). Not available on Abstract, Plasma, Linea, Hyper, Ink, BSC, Avalanche, Polygon. See [chain-feature-matrix](../docs/chain-feature-matrix.md).
- Creator fee claiming; optional fee recipient; gas sponsored within limits
- Rate limits: 1/day standard, 10/day Torque Club (gas sponsored within limits)

**Reference**: [references/token-deployment.md](references/token-deployment.md)

### Automation

- Limit orders
- Stop loss orders
- DCA (dollar-cost averaging)
- TWAP (time-weighted average price)
- Scheduled commands

**Reference**: [references/automation.md](references/automation.md)

### Arbitrary Transactions

- Submit raw EVM transactions with explicit calldata
- Custom contract calls to any address
- Execute pre-built calldata from other tools
- Value transfers with data

**Reference**: [references/arbitrary-transaction.md](references/arbitrary-transaction.md)

## Supported Chains

Single source of truth: [docs/chains-config.md](../docs/chains-config.md). **Chain requirements:** Some features are limited to a subset of chains; see [chain-feature-matrix](../docs/chain-feature-matrix.md).

| Chain     | Chain ID | Native Token |
|-----------|----------|--------------|
| Ethereum  | 1        | ETH          |
| BSC       | 56       | BNB          |
| Polygon   | 137      | MATIC        |
| Monad     | 143      | MON          |
| Arbitrum  | 42161    | ETH          |
| Avalanche | 43114    | AVAX         |
| Base      | 8453     | ETH          |
| Plasma    | 9745     | ETH          |
| Hyper     | 999      | HYPE         |
| Abstract  | 2741     | ABS          |
| Linea     | 59144    | ETH          |
| Ink       | 57073    | INK          |

## Safety & Access Control

**Dedicated Agent Wallet**: When building autonomous agents, create a separate Torque account rather than using your personal wallet. This isolates agent funds — if a key is compromised, only the agent wallet is exposed. Fund it with limited amounts and replenish as needed.

**API Key Types**: Torque uses a single key format (`bk_...`) with capability flags (`agentApiEnabled`, `llmGatewayEnabled`). You can optionally configure a separate LLM Gateway key via `torque config set llmKey` or `TORQUE_LLM_KEY` — useful when you want independent revocation or different permissions for agent vs LLM access.

**Read-Only API Keys**: Keys with `readOnly: true` filter all write tools (swaps, transfers, staking, token launches, etc.) from agent sessions. The `/agent/sign` and `/agent/submit` endpoints return 403. Ideal for monitoring bots and research agents.

**IP Whitelisting**: Set `allowedIps` on your API key to restrict usage to specific IPs. Requests from non-whitelisted IPs are rejected with 403 at the auth layer.

**Rate Limits**: 100 messages/day (standard), 1,000/day (Torque Club), or custom per key. Resets 24h from first message (rolling window). LLM Gateway uses a credit-based system.

**Key safety rules:**
- Store keys in environment variables (`TORQUE_API_KEY`, `TORQUE_LLM_KEY`), never in source code
- Add `~/.torque/` and `.env` to `.gitignore` — the CLI stores credentials in `~/.torque/config.json`
- Test with small amounts on low-cost chains (Base, Polygon) before production use
- Use `waitForConfirmation: true` with `/agent/submit` — transactions execute immediately with no confirmation prompt
- Rotate keys periodically and revoke immediately if compromised at [app.torque.fi/api](https://app.torque.fi/api)

**Reference**: [references/safety.md](references/safety.md)

## Common Patterns

### Check Before Trading

```bash
# Check balance
torque prompt "What is my ETH balance on Base?"

# Check price
torque prompt "What's the current price of PEPE?"

# Then trade
torque prompt "Buy $20 of PEPE on Base"
```

### Portfolio Review

```bash
# Full portfolio
torque prompt "Show my complete portfolio"

# Chain-specific
torque prompt "What tokens do I have on Base?"

# Token-specific
torque prompt "Show my ETH across all chains"
```

### Set Up Automation

```bash
# DCA strategy
torque prompt "DCA $100 into ETH every week"

# Stop loss protection
torque prompt "Set stop loss for my ETH at $2,500"

# Limit order
torque prompt "Buy ETH if price drops to $3,000"
```

### Market Research

```bash
# Price and analysis
torque prompt "Do technical analysis on ETH"

# Trending tokens
torque prompt "What tokens are trending on Base?"

# Compare tokens
torque prompt "Compare ETH vs SOL"
```

## API Workflow

Torque uses an asynchronous job-based API:

1. **Submit** — Send prompt (with optional `threadId`), get job ID and thread ID
2. **Poll** — Check status every 2 seconds
3. **Complete** — Process results when done
4. **Continue** — Reuse `threadId` for multi-turn conversations

The `torque prompt` command handles this automatically. When using the REST API directly, implement the poll loop yourself (see Option 2 above or the reference below). For manual job control via CLI, use `torque status <jobId>` and `torque cancel <jobId>`.

For details on the API structure, job states, polling strategy, and error handling, see:

**Reference**: [references/api-workflow.md](references/api-workflow.md)

### Synchronous Endpoints

For direct signing and transaction submission, Torque also provides synchronous endpoints:

- **POST /agent/sign** - Sign messages, typed data, or transactions without broadcasting
- **POST /agent/submit** - Submit raw transactions directly to the blockchain

These endpoints return immediately (no polling required) and are ideal for:
- Authentication flows (sign messages)
- Gasless approvals (sign EIP-712 permits)
- Pre-built transactions (submit raw calldata)

**Reference**: [references/sign-submit-api.md](references/sign-submit-api.md)

## Error Handling

Common issues and fixes:

- **Authentication errors** → Run `torque login` or check `torque whoami` (CLI), or verify your `X-API-Key` header (REST API)
- **Insufficient balance** → Add funds or reduce amount
- **Token not found** → Verify symbol and chain
- **Transaction reverted** → Check parameters and balances
- **Rate limiting** → Wait and retry

For comprehensive error troubleshooting, setup instructions, and debugging steps, see:

**Reference**: [references/error-handling.md](references/error-handling.md)

## Best Practices

### Security

1. Never share your API key or LLM key
2. Use a dedicated agent wallet with limited funds for autonomous agents
3. Use read-only API keys for monitoring and research-only agents
4. Set IP whitelisting for server-side agents with known IPs
5. Verify addresses before large transfers
6. Use stop losses for leverage trading
7. Store keys in environment variables, not source code — add `~/.torque/` to `.gitignore`

See [references/safety.md](references/safety.md) for comprehensive safety guidance.

### Trading

1. Check balance before trades
2. Specify chain for lesser-known tokens
3. Consider gas costs (use Base/Polygon for small amounts)
4. Start small, scale up after testing
5. Use limit orders for better prices

### Automation

1. Test automation with small amounts first
2. Review active orders regularly
3. Set realistic price targets
4. Always use stop loss for leverage
5. Monitor execution and adjust as needed

## Tips for Success

### For New Users

- Start with balance checks and price queries
- Test with $5-10 trades first
- Use Base for lower fees
- Enable trading confirmations initially
- Learn one feature at a time

### For Experienced Users

- Leverage automation for strategies
- Use multiple chains for diversification
- Combine DCA with stop losses
- Explore advanced features (leverage, Polymarket)
- Monitor gas costs across chains

## Prompt Examples by Category

### Trading

- "Buy $50 of ETH on Base"
- "Swap 0.1 ETH for USDC"
- "Sell 50% of my PEPE"
- "Bridge 100 USDC from Polygon to Base"

### Portfolio

- "Show my portfolio"
- "What's my ETH balance?"
- "Total portfolio value"
- "Holdings on Base"

### Market Research

- "What's the price of Bitcoin?"
- "Analyze ETH price"
- "Trending tokens on Base"
- "Compare UNI vs SUSHI"

### Transfers

- "Send 0.1 ETH to vitalik.eth"
- "Transfer $20 USDC to @friend"
- "Send 50 USDC to 0x123..."

### NFTs

- "Show Bored Ape floor price"
- "Buy cheapest Pudgy Penguin"
- "Show my NFTs"

### Polymarket

Polymarket is on **Polygon only** (see [chain-feature-matrix](../docs/chain-feature-matrix.md)).

- "What are the odds Trump wins?"
- "Bet $10 on Yes for [market]"
- "Show my Polymarket positions"

### Leverage

- "Open 5x long on ETH with $100"
- "Short BTC 10x with stop loss at $45k"
- "Show my Avantis positions"

### Automation

- "DCA $100 into ETH weekly"
- "Set limit order to buy ETH at $3,000"
- "Stop loss for all holdings at -20%"

### Token Deployment

**EVM (Clanker):** Only on Base, Ethereum, Arbitrum, and Monad (see [chain-feature-matrix](../docs/chain-feature-matrix.md)).

- "Deploy a token called TorqueFan with symbol BFAN on Base"
- "Deploy a token on Arbitrum" / "… on Monad"
- "Claim fees for my token MTK"

### Arbitrary Transactions

- "Submit this transaction: {to: 0x..., data: 0x..., value: 0, chainId: 8453}"
- "Execute this calldata on Base: {...}"
- "Send raw transaction with this JSON: {...}"

### Sign API (Synchronous)

Direct message signing without AI processing:

```bash
# Sign a plain text message
curl -X POST "https://api.torque.fi/agent/sign" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"signatureType": "personal_sign", "message": "Hello, Torque!"}'

# Sign EIP-712 typed data (permits, orders)
curl -X POST "https://api.torque.fi/agent/sign" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"signatureType": "eth_signTypedData_v4", "typedData": {...}}'

# Sign a transaction without broadcasting
curl -X POST "https://api.torque.fi/agent/sign" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"signatureType": "eth_signTransaction", "transaction": {"to": "0x...", "chainId": 8453}}'
```

### Submit API (Synchronous)

Direct transaction submission without AI processing:

```bash
# Submit a raw transaction
curl -X POST "https://api.torque.fi/agent/submit" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "transaction": {"to": "0x...", "chainId": 8453, "value": "1000000000000000000"},
    "waitForConfirmation": true
  }'
```

**Reference**: [references/sign-submit-api.md](references/sign-submit-api.md)

## Resources

- **Documentation**: https://docs.torque.fi
- **LLM Gateway Docs**: https://docs.torque.fi/llm-gateway/overview
- **API Key Management**: https://app.torque.fi/api
- **Assistant**: https://app.torque.fi/assistant
- **CLI Package**: https://www.npmjs.com/package/@torque/cli
- **Twitter**: @torque_bot

## Troubleshooting

### CLI Not Found

```bash
# Verify installation
which torque

# Reinstall if needed
bun install -g @torque/cli
```

### Authentication Issues

**CLI:**
```bash
# Check current auth
torque whoami

# Re-authenticate
torque login

# Check LLM key specifically
torque config get llmKey
```

**REST API:**
```bash
# Test your API key
curl -s "https://api.torque.fi/_health" -H "X-API-Key: $TORQUE_API_KEY"
```

### API Errors

See [references/error-handling.md](references/error-handling.md) for comprehensive troubleshooting.

### Getting Help

1. Check error message in CLI output or API response
2. Run `torque whoami` to verify auth (CLI) or test with a curl to `/_health` (REST API)
3. Consult relevant reference document
4. Test with simple queries first (`torque prompt "What is my balance?"` or `POST /agent/prompt`)

---

**Pro Tip**: The most common issue is not specifying the chain for tokens. When in doubt, always include "on Base" or "on Ethereum" in your prompt.

**Security**: Keep your API key private. Never commit your config file to version control. Only trade amounts you can afford to lose.

**Quick Win**: Start by checking your portfolio (`torque prompt "Show my portfolio"`) to see what's possible, then try a small $5-10 trade on Base to get familiar with the flow.
