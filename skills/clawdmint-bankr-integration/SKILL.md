---
name: Clawdmint - Bankr Integration
description: This skill should be used when the user asks about "Bankr and Clawdmint", "use Bankr to mint NFT", "Bankr SDK Clawdmint", "deploy NFT with Bankr", "Bankr agent NFT", "natural language NFT deployment", "Bankr x402 Clawdmint", or any integration between Bankr SDK/API and the Clawdmint NFT platform.
version: 1.0.0
---

# Clawdmint + Bankr Integration

Deploy and manage NFT collections on Clawdmint using Bankr's natural language AI interface and SDK.

## Overview

Bankr provides two integration paths with Clawdmint:

1. **Bankr Agent API** — Natural language prompts via REST API (job-based async)
2. **@bankr/sdk** — TypeScript SDK with x402 micropayments ($0.01/request)

Both support Clawdmint's NFT deployment, collection browsing, and management.

## Method 1: Bankr Agent API

### Setup

```bash
export BANKR_API_KEY=bk_your_api_key
```

### Deploy Collection via Natural Language

```typescript
const BANKR_API = "https://api.bankr.bot";

// Submit deployment prompt
const submitRes = await fetch(`${BANKR_API}/agent/prompt`, {
  method: "POST",
  headers: {
    "X-API-Key": process.env.BANKR_API_KEY,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    prompt: "Deploy an NFT collection on Clawdmint called 'Robot Dreams' with symbol RBOT, 100 max supply, 0.001 ETH mint price"
  }),
});

const { jobId } = await submitRes.json();

// Poll for result
async function pollJob(jobId: string): Promise<any> {
  while (true) {
    const res = await fetch(`${BANKR_API}/agent/job/${jobId}`, {
      headers: { "X-API-Key": process.env.BANKR_API_KEY }
    });
    const job = await res.json();

    if (job.status === "completed") return job;
    if (job.status === "failed") throw new Error(job.error);

    // Report progress
    if (job.statusUpdates?.length) {
      console.log("Progress:", job.statusUpdates.at(-1));
    }

    await new Promise(r => setTimeout(r, 2000)); // Poll every 2s
  }
}

const result = await pollJob(jobId);
console.log("Result:", result.response);
```

### Natural Language Prompts

| Intent | Example Prompt |
|--------|----------------|
| Deploy | "Deploy NFT collection 'Cyber Art' on Clawdmint, 50 supply, 0.002 ETH" |
| Browse | "Show me active NFT collections on Clawdmint" |
| Check | "What's the status of my Clawdmint agent?" |
| Stats | "How many NFTs have been minted on Clawdmint?" |
| Price | "How much does it cost to deploy on Clawdmint?" |

## Method 2: @bankr/sdk (x402)

### Setup

```bash
npm install @bankr/sdk
export BANKR_PRIVATE_KEY=0x...  # Needs USDC on Base
```

### Deploy with SDK

```typescript
import { BankrClient } from "@bankr/sdk";

const client = new BankrClient({
  privateKey: process.env.BANKR_PRIVATE_KEY as `0x${string}`,
  walletAddress: process.env.BANKR_WALLET_ADDRESS, // optional
});

// Deploy via natural language
const result = await client.promptAndWait({
  prompt: "Deploy an NFT collection on Clawdmint: name='Bankr Genesis', symbol='BGEN', max supply 200, mint price 0.0005 ETH, royalty 5%"
});

if (result.status === "completed") {
  console.log(result.response);

  // If transactions are returned, execute them
  if (result.transactions?.length) {
    for (const tx of result.transactions) {
      console.log(`Execute tx: ${tx.type}`, tx.metadata.transaction);
    }
  }
}
```

### Browse Collections

```typescript
const collections = await client.promptAndWait({
  prompt: "List all public NFT collections on Clawdmint with their mint prices"
});
console.log(collections.response);
```

### Check Agent Status

```typescript
const status = await client.promptAndWait({
  prompt: "Check my Clawdmint agent verification status"
});
console.log(status.response);
```

## Direct x402 Integration (Without Bankr)

You can also call Clawdmint's x402 endpoints directly:

```typescript
import { x402Fetch } from "@x402/fetch";

// $2.00 USDC — Deploy collection
const deployRes = await x402Fetch(
  "https://clawdmint.xyz/api/x402/deploy",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      name: "Direct Deploy",
      symbol: "DDEP",
      image: "https://example.com/cover.png",
      max_supply: 100,
      mint_price_eth: "0.001",
      payout_address: "0xYourWallet",
    }),
  },
  { wallet: myWalletClient }
);

// $0.001 USDC — List collections
const listRes = await x402Fetch(
  "https://clawdmint.xyz/api/x402/collections",
  { method: "GET" },
  { wallet: myWalletClient }
);
```

## Building a Bankr-Powered Clawdmint Bot

Complete example of a bot that auto-deploys collections:

```typescript
import { BankrClient } from "@bankr/sdk";

const client = new BankrClient({
  privateKey: process.env.BANKR_PRIVATE_KEY as `0x${string}`,
});

async function autoDeployCollection(theme: string) {
  // Step 1: Generate collection concept
  const concept = await client.promptAndWait({
    prompt: `Generate a creative NFT collection concept with theme "${theme}". Include a name, symbol, description, and suggest a mint price.`
  });

  // Step 2: Deploy on Clawdmint
  const deploy = await client.promptAndWait({
    prompt: `Deploy this NFT collection on Clawdmint: ${concept.response}`
  });

  // Step 3: Announce
  console.log("New collection deployed!");
  console.log(deploy.response);

  return deploy;
}

// Deploy a collection with "cyberpunk" theme
autoDeployCollection("cyberpunk").catch(console.error);
```

## Environment Variables

| Variable | Where | Description |
|----------|-------|-------------|
| `BANKR_API_KEY` | Bankr Agent API | Your `bk_` prefixed API key |
| `BANKR_PRIVATE_KEY` | @bankr/sdk | Private key (pays $0.01 USDC/request) |
| `BANKR_WALLET_ADDRESS` | @bankr/sdk | Optional wallet for receiving tokens |
| `CLAWDMINT_API_KEY` | Clawdmint direct | For direct API calls (not needed with Bankr) |

## Cost Comparison

| Method | Deploy Cost | Per-Query Cost |
|--------|------------|----------------|
| Clawdmint API Key | Free (after registration) | Free |
| Clawdmint x402 Direct | $2.00 USDC | $0.001 - $0.005 |
| Bankr Agent API | API key based | Free with API key |
| @bankr/sdk | $0.01 USDC + $2.00 deploy | $0.01 per query |

## Common Issues

| Issue | Solution |
|-------|----------|
| "Payment required" | Ensure USDC balance on Base |
| Job stuck in "processing" | Poll for 2-3 min, then cancel and retry |
| Collection not found | Clawdmint may not be in Bankr's known integrations yet |
| SDK timeout | Increase timeout or use manual polling pattern |
