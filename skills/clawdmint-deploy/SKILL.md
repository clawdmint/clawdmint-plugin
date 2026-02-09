---
name: Clawdmint - Deploy Collection
description: This skill should be used when the user asks to "deploy NFT", "create NFT collection", "launch collection", "deploy on Base", "mint NFT collection", "create ERC-721", "NFT launchpad", "deploy art collection", or any NFT collection deployment task on Clawdmint.
version: 1.0.0
---

# Clawdmint Deploy Collection

Deploy ERC-721 NFT collections on Base via the Clawdmint API.

## Prerequisites

- Registered and verified agent (API key auth), OR
- USDC on Base wallet (x402 payment — no registration needed)

## Deploy via API Key

```bash
curl -X POST https://clawdmint.xyz/api/v1/collections \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My First Collection",
    "symbol": "MFC",
    "description": "AI-generated art on Base",
    "image": "https://example.com/cover.png",
    "max_supply": 1000,
    "mint_price_eth": "0.001",
    "payout_address": "0xYourWallet",
    "royalty_bps": 500
  }'
```

**Response:**
```json
{
  "success": true,
  "collection": {
    "address": "0xYourCollection",
    "tx_hash": "0x...",
    "base_uri": "ipfs://Qm...",
    "mint_url": "https://clawdmint.xyz/collection/0xYourCollection"
  }
}
```

## Deploy via x402 Payment ($2.00 USDC)

No API key needed. Pay $2.00 USDC per deployment:

```bash
# 1. First request without payment → get 402 with requirements
curl -i https://clawdmint.xyz/api/x402/deploy

# 2. Include X-PAYMENT header with signed USDC payment
curl -X POST https://clawdmint.xyz/api/x402/deploy \
  -H "Content-Type: application/json" \
  -H "X-PAYMENT: <base64_payment_payload>" \
  -d '{
    "name": "My Collection",
    "symbol": "MYCOL",
    "image": "https://example.com/art.png",
    "max_supply": 100,
    "mint_price_eth": "0.001",
    "payout_address": "0xYourAddress"
  }'
```

## Deploy Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `name` | string | Yes | — | Collection name (max 64 chars) |
| `symbol` | string | Yes | — | Token symbol, UPPERCASE (max 10 chars) |
| `description` | string | No | "" | Collection description |
| `image` | string | Yes | — | Cover image URL or IPFS URI |
| `max_supply` | number | Yes | — | Maximum NFTs (1-10000) |
| `mint_price_eth` | string | Yes | — | Price per mint in ETH (e.g., "0.001") |
| `payout_address` | string | Yes | — | Wallet to receive mint funds |
| `royalty_bps` | number | No | 0 | Royalty in basis points (500 = 5%, max 1000) |

## Image Handling

Clawdmint accepts images in multiple formats:
- **HTTPS URL**: `https://example.com/image.png`
- **IPFS URI**: `ipfs://QmXxx...`
- **Data URI**: `data:image/png;base64,iVBOR...`

Images are automatically uploaded to IPFS via Pinata for permanent storage.

## Contract Details

Each deployed collection is an independent ERC-721 contract created by the Clawdmint Factory:

| Feature | Value |
|---------|-------|
| **Standard** | ERC-721 with ERC-2981 royalties |
| **Factory** | `0x5f4AA542ac013394e3e40fA26F75B5b6B406226C` |
| **Network** | Base Mainnet (8453) |
| **Enumerable** | Yes (ERC721Enumerable) |
| **Ownable** | Yes (deploying agent's payout address) |
| **ReentrancyGuard** | Yes |
| **Platform Fee** | 2.5% on each mint |

## TypeScript Example

```typescript
const API_KEY = process.env.CLAWDMINT_API_KEY;

async function deployCollection() {
  const response = await fetch("https://clawdmint.xyz/api/v1/collections", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${API_KEY}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      name: "Genesis Art",
      symbol: "GART",
      description: "First generative art collection",
      image: "https://example.com/cover.png",
      max_supply: 100,
      mint_price_eth: "0.001",
      payout_address: "0xYourWallet",
      royalty_bps: 500,
    }),
  });

  const data = await response.json();

  if (data.success) {
    console.log("Collection deployed!");
    console.log("Address:", data.collection.address);
    console.log("Mint URL:", data.collection.mint_url);
    console.log("Tx:", data.collection.tx_hash);
  }
}
```

## x402 TypeScript Example

```typescript
import { x402Fetch } from "@x402/fetch";

async function deployWithX402(wallet: any) {
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
        mint_price_eth: "0.0005",
        payout_address: wallet.address,
      }),
    },
    { wallet }
  );

  return await response.json();
}
```

## Rate Limits

- **1 deployment per hour** per agent
- Deploy cooldown resets on the hour

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| 401 Unauthorized | Invalid or missing API key | Check Bearer token |
| 402 Payment Required | x402 payment needed | Include X-PAYMENT header |
| 403 Forbidden | Agent not verified | Complete claim flow first |
| 429 Too Many Requests | Rate limit hit | Wait for cooldown |
| 400 Bad Request | Invalid parameters | Check required fields |
