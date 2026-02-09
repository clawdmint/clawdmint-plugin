# Clawdmint Claude Plugin

Deploy NFT collections on Base with AI agents. Supports API key authentication, x402 USDC micropayments, and Bankr SDK integration.

## Features

- **NFT Deployment**: Deploy ERC-721 collections on Base via API or x402 payment
- **Agent Registration**: Register and verify AI agent identities
- **Collection Management**: Browse, query, and monitor NFT collections
- **x402 Payments**: Pay-per-request with USDC on Base (no registration needed)
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
    "mint_price_eth": "0.001",
    "payout_address": "0xYourWallet"
  }'
```

### Or Deploy with x402 ($2.00 USDC, no registration)

```typescript
import { x402Fetch } from "@x402/fetch";

const response = await x402Fetch(
  "https://clawdmint.xyz/api/x402/deploy",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      name: "x402 Collection",
      symbol: "X402",
      image: "https://example.com/art.png",
      max_supply: 50,
      mint_price_eth: "0.001",
      payout_address: wallet.address,
    }),
  },
  { wallet }
);
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `CLAWDMINT_API_KEY` | For API auth | Agent API key from registration |
| `BANKR_API_KEY` | For Bankr API | Bankr API key (prefix: `bk_`) |
| `BANKR_PRIVATE_KEY` | For SDK/x402 | Wallet private key (needs USDC on Base) |

## Technical Specs

| Spec | Value |
|------|-------|
| **Network** | Base Mainnet (Chain ID 8453) |
| **Factory** | `0x5f4AA542ac013394e3e40fA26F75B5b6B406226C` |
| **NFT Standard** | ERC-721 + EIP-2981 |
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
