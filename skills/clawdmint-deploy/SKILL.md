---
name: Clawdmint - Deploy Collection
description: This skill should be used when the user asks to "deploy NFT", "create NFT collection", "launch collection", "deploy on Solana", "mint NFT collection", "NFT launchpad", "deploy art collection", or any NFT collection deployment task on Clawdmint.
version: 1.0.0
---

# Clawdmint Deploy Collection

Deploy Solana NFT collections via the Clawdmint API.

## Prerequisites

- Registered and verified agent (API key auth), OR
- Solana USDC payment for x402-gated surfaces

## Deploy via API Key

```bash
curl -X POST https://clawdmint.xyz/api/v1/collections \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My First Collection",
    "symbol": "MFC",
    "description": "AI-generated art on Solana",
    "image": "https://example.com/cover.png",
    "max_supply": 1000,
    "mint_price_sol": "0.05",
    "payout_address": "SellerWalletBase58",
    "royalty_bps": 500
  }'
```

**Response:**
```json
{
  "success": true,
  "collection": {
    "address": "SolanaCollectionAddress",
    "tx_hash": "SolanaSignature",
    "base_uri": "ipfs://Qm...",
    "mint_url": "https://clawdmint.xyz/collection/0xYourCollection"
  }
}
```

## Deploy via x402 Payment ($2.00 USDC)

Pay $2.00 USDC on Solana before the deployment request is processed:

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
    "mint_price_sol": "0.05",
    "payout_address": "SellerWalletBase58"
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
| `mint_price_sol` | string | Yes | — | Price per mint in SOL (e.g., "0.05") |
| `payout_address` | string | Yes | — | Wallet to receive mint funds |
| `royalty_bps` | number | No | 0 | Royalty in basis points (500 = 5%, max 1000) |

## Image Handling

Clawdmint accepts images in multiple formats:
- **HTTPS URL**: `https://example.com/image.png`
- **IPFS URI**: `ipfs://QmXxx...`
- **Data URI**: `data:image/png;base64,iVBOR...`

Images are automatically uploaded to IPFS via Pinata for permanent storage.

## Collection Details

Each deployed collection follows the configured Solana collection flow:

| Feature | Value |
|---------|-------|
| **Standard** | Solana NFT collection flow |
| **Network** | Solana |
| **Authority** | Agent wallet or configured collection authority |
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
      mint_price_sol: "0.05",
      payout_address: "SellerWalletBase58",
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

## Solana x402 Client Note

Read `PAYMENT-REQUIRED`, sign a matching Solana USDC transfer transaction, and retry with `X-PAYMENT`. See `clawdmint-x402-payments` for the exact header format.

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
