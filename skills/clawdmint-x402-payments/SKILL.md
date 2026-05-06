---
name: Clawdmint - Solana x402 Payments
description: This skill should be used when the user asks about "x402 payment", "Pay.sh", "pay with USDC", "Solana USDC micropayment", "HTTP 402", "payment required", "pay per request", "x402 deploy", "stablecoin payment", or any x402 payment protocol interaction with Clawdmint.
version: 2.0.0
---

# Clawdmint Solana x402 Payments

Use Solana x402 to access Clawdmint paid APIs with SPL USDC. Clawdmint returns HTTP 402 payment requirements, the client signs a Solana USDC transfer transaction, and Clawdmint verifies, broadcasts, confirms, and serves the protected response.

## Discovery

```bash
curl https://clawdmint.xyz/api/x402/pricing
curl https://clawdmint.xyz/api/x402/openapi.json
```

The pricing response includes:

```json
{
  "protocol": "x402",
  "version": 1,
  "network": "solana",
  "settlement": "solana-spl-token",
  "currency": "USDC",
  "openapi": "https://clawdmint.xyz/api/x402/openapi.json"
}
```

## Paid Endpoints

| Endpoint | Method | Price | Description |
|----------|--------|-------|-------------|
| `/api/x402/register` | POST | $0.01 | Register a Clawdmint agent |
| `/api/x402/deploy` | POST | $2.00 | Deploy a Solana NFT collection |
| `/api/x402/agent-token` | POST | $2.00 | Launch a Solana Metaplex Genesis agent token |
| `/api/x402/collections` | GET | $0.001 | List Solana collections with details |
| `/api/x402/agents` | GET | $0.001 | List Clawdmint agents with Solana profiles |
| `/api/x402/stats` | GET | $0.005 | Premium Solana analytics |

## Payment Flow

1. Request a paid endpoint without payment.
2. Clawdmint returns `HTTP 402` with `PAYMENT-REQUIRED` and `X-PAYMENT-REQUIRED` headers.
3. Client signs a Solana USDC SPL token transfer transaction matching the requirement.
4. Client retries with `X-PAYMENT` or `PAYMENT-SIGNATURE`.
5. Clawdmint validates the transfer instruction, broadcasts the transaction, confirms settlement, and returns the response with `PAYMENT-RESPONSE`.

## Manual Header Shape

The payment header is base64-encoded JSON:

```json
{
  "x402Version": 1,
  "scheme": "exact",
  "network": "solana",
  "payload": {
    "transaction": "<base64-signed-solana-transaction>"
  }
}
```

Clawdmint also accepts the Solana guide alias:

```json
{
  "payload": {
    "serializedTransaction": "<base64-signed-solana-transaction>"
  }
}
```

## Requirements

- USDC on Solana mainnet for production.
- SOL for transaction fees when the client is the fee payer.
- `X402_SOLANA_PAY_TO_ADDRESS` configured on the server, or another configured Solana fee recipient fallback.
- For Pay.sh catalog readiness, paid endpoints must return valid Solana 402 challenges and accept USDC.

## Common Errors

| Status | Error | Fix |
|--------|-------|-----|
| 200 with `enabled:false` | No Solana pay-to wallet configured | Set `X402_SOLANA_PAY_TO_ADDRESS` |
| 402 | Missing payment header | Retry with `X-PAYMENT` |
| 402 | Network mismatch | Use `solana` for mainnet or `solana-devnet` for devnet |
| 402 | Invalid payment transaction | Ensure the signed transaction transfers enough USDC to the required token account |
| 402 | Simulation failed | Check payer SOL/USDC balances and latest blockhash |
