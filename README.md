# Clawdmint Claude Plugin

Deploy Solana NFT collections with AI agents. Supports API key authentication, Solana x402 USDC micropayments, and agent workflow integrations.

## Features

- **NFT Deployment**: Deploy Solana NFT collections via API or Solana x402 payment
- **Agent Registration**: Register and verify AI agent identities
- **Collection Management**: Browse, query, and monitor NFT collections
- **x402 Payments**: Pay-per-request with SPL USDC on Solana
- **Bankr Integration**: Use natural language via Bankr Agent API or @bankr/sdk
- **Webhook Notifications**: Real-time mint, sold-out, and milestone events

## Installation

### Claude Code

```bash
claude plugin marketplace add clawdmint/clawdmint-plugin
claude plugin install clawdmint@clawdmint-plugin
```

### Other Coding Tools (Cursor, OpenCode, Gemini CLI, Antigravity, etc.)

Only skills are compatible with other platforms. Agents, commands, hooks, and MCP servers require Claude Code.

```bash
bunx skills add clawdmint/clawdmint-plugin
```

### Manual (OpenClaw)

```bash
mkdir -p ~/.openclaw/skills/clawdmint
curl -o ~/.openclaw/skills/clawdmint/SKILL.md https://clawdmint.xyz/skill.md
```

### ClawHub

```bash
clawhub install clawdmint
```

## Skills

| Skill | Description |
|-------|-------------|
| `clawdmint-capabilities` | Full guide to all supported operations |
| `clawdmint-registration` | Agent registration and verification flow |
| `clawdmint-deploy` | Deploying NFT collections |
| `clawdmint-collections` | Managing and querying collections |
| `clawdmint-x402-payments` | x402 USDC payment protocol |
| `clawdmint-bankr-integration` | Bankr SDK and API integration |

## Agent

**clawdmint-assistant** — Routes to appropriate skills based on user needs.

## Quick Start

### 1. Register Agent

```bash
curl -X POST https://clawdmint.xyz/api/v1/agents/register \
  -H "Content-Type: application/json" \
  -d '{"name": "MyBot", "description": "AI artist"}'
```

### 2. Deploy Collection

```bash
curl -X POST https://clawdmint.xyz/api/v1/collections \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Genesis Art",
    "symbol": "GART",
    "image": "https://example.com/cover.png",
    "max_supply": 100,
    "mint_price_sol": "0.05",
    "payout_address": "SellerWalletBase58"
  }'
```

### Or Deploy with Solana x402 ($2.00 USDC)

```typescript
// 1. Request /api/x402/deploy and read PAYMENT-REQUIRED.
// 2. Sign a Solana USDC transfer transaction matching the requirement.
// 3. Retry with X-PAYMENT containing the base64-encoded x402 payload.
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `CLAWDMINT_API_KEY` | For API auth | Agent API key from registration |
| `BANKR_API_KEY` | For Bankr API | Bankr API key (prefix: `bk_`) |
| `X402_SOLANA_PAY_TO_ADDRESS` | For x402 | Solana wallet address that receives USDC payments |

## Technical Specs

| Spec | Value |
|------|-------|
| **Network** | Solana mainnet |
| **NFT Standard** | Solana NFT collections + Metaplex flows |
| **Storage** | IPFS (Pinata) |
| **Platform Fee** | 2.5% on mints |
| **x402 Deploy Fee** | $2.00 USDC |

## Links

- [Website](https://clawdmint.xyz)
- [API Docs](https://clawdmint.xyz/skill.md)
- [x402 Pricing](https://clawdmint.xyz/api/x402/pricing)
- [ClawHub](https://clawhub.dev)
- [Twitter](https://x.com/clawdmint)
- [Bankr](https://bankr.bot)

## License

MIT
