# Billing

JarvisClaw supports two billing modes: x402 per-call payments and pre-paid API key balance.

## Billing Modes

### x402 Per-Call Payment

Pay for each API call directly with USDC from your wallet. No account or pre-funding needed.

- **Chains**: Base or Solana
- **Token**: USDC
- **Pricing**: Charged per request based on token usage
- **Settlement**: On-chain, verified by facilitator

Best for: AI agents, pay-as-you-go usage, no upfront commitment.

### API Key Pre-Paid

Fund your account balance and consume it with API key auth.

- **Top-up**: Dashboard or USDC deposit
- **Auth**: Bearer token (`sk-...`)
- **Pricing**: Deducted from balance per request based on token usage

Best for: Predictable workloads, team usage, rate limit customization.

## Pricing

Model pricing follows upstream provider rates. Costs are calculated per token (input and output counted separately for LLMs) or per image/video/audio unit.

Use the dashboard to view real-time costs per request in your usage logs.

## Daily Limits

Accounts have configurable daily spending limits to prevent runaway costs:

| Setting | Default |
|---------|---------|
| Daily limit | Set per account in dashboard |
| Per-request max | Based on model pricing |
| Rate limit | Requests per minute per key |

When a daily limit is reached, requests return HTTP 429 until the next UTC day.

## Checking Balance

::: code-group

```python [Python]
from jarvisclaw import Client

client = Client(api_key="sk-your-api-key")
balance = client.balance()
print(f"Remaining: ${balance.available}")
```

```bash [cURL]
curl https://api.jarvisclaw.ai/v1/balance \
  -H "Authorization: Bearer sk-your-api-key"
```

:::
