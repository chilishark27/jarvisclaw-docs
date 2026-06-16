# Solana Payments (x402)

Pay for API calls with USDC on Solana — no API key or account required.

## Overview

JarvisClaw supports x402 payments on Solana using SPL USDC. The SDK automatically detects Solana keys from their base58 format (as opposed to hex `0x...` keys used for Base chain).

## How It Works

1. Your agent sends a request without auth
2. Server responds HTTP 402 with payment requirements
3. Agent signs an x402 payment using its Solana keypair
4. Agent resends the request with the `PAYMENT-SIGNATURE` header
5. Server verifies payment via the facilitator and returns the response

## Key Detection

The SDK detects the chain automatically from key format:

| Key Format | Chain | Example |
|-----------|-------|---------|
| Hex (`0x...`) | Base (EVM) | `0x4c0883a6...` |
| Base58 (64-byte keypair) | Solana | `5K7gBTk8nM...` |

No manual chain configuration needed.

## Python SDK

```python
from jarvisclaw import AgentClient

# Solana key — auto-detected from base58 format
client = AgentClient(private_key="your_solana_base58_keypair")

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello from Solana!"}],
)
print(response.choices[0].message.content)
```

## Go SDK

```go
// Solana keypair — detected automatically
client := jc.New(jc.WithPrivateKey("your_solana_base58_keypair"))

resp, err := client.Chat(ctx, &jc.ChatRequest{
    Model:    "gpt-4o",
    Messages: []jc.Message{{Role: "user", Content: "Hello from Solana!"}},
})
```

## Payment Details

| Parameter | Value |
|-----------|-------|
| Network | Solana |
| Asset | USDC (SPL Token) |
| Facilitator | `https://api.cdp.coinbase.com/platform/v2/x402` |
| Protocol | x402 v2 |

## Getting a Solana Wallet

Any Solana wallet that exposes a keypair works:

- **Phantom** — export private key from settings
- **Solflare** — export keypair
- **solana-keygen** — `solana-keygen new --outfile key.json`

Ensure your wallet holds USDC (SPL) on Solana mainnet. You can bridge USDC from other chains using standard bridges.

## Comparison with Base

Both chains are fully supported. Choose based on preference:

| | Base | Solana |
|---|---|---|
| Key format | Hex (0x...) | Base58 |
| Token | USDC (ERC-20) | USDC (SPL) |
| Finality | ~2s | ~400ms |
| Gas fees | Very low | Very low |
