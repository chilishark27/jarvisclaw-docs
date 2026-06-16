# Discovery Protocol

JarvisClaw exposes a standard x402 discovery endpoint that any agent can query to learn about available services and payment methods.

## Discovery Endpoint

```bash
curl https://api.jarvisclaw.ai/.well-known/x402/discovery
```

No authentication required. Returns:

```json
{
  "x402Version": 2,
  "provider": {
    "name": "JarvisClaw",
    "description": "AI API routing platform with smart routing, pay per call via x402",
    "url": "https://api.jarvisclaw.ai",
    "docs": "https://docs.jarvisclaw.ai"
  },
  "paymentInfo": {
    "protocol": "x402",
    "version": "2",
    "networks": ["eip155:8453", "solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp"],
    "assets": {
      "eip155:8453": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
      "solana:...": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"
    },
    "payTo": { "eip155:8453": "0x...", "solana:...": "..." },
    "facilitator": "https://api.cdp.coinbase.com/platform/v2/x402"
  },
  "resources": [...],
  "capabilities": {
    "streaming": true,
    "multimodal": true,
    "functionCalling": true,
    "openaiCompatible": true,
    "claudeCompatible": true,
    "geminiCompatible": true
  },
  "authentication": {
    "x402": { "header": "PAYMENT-SIGNATURE" },
    "apiKey": { "header": "Authorization", "format": "Bearer sk-..." }
  }
}
```

## Resources

The discovery response lists available resource types:

| Type | Endpoint | Description |
|------|----------|-------------|
| `ai-model` | `/v1/chat/completions` | Chat completions (per-token billing) |
| `ai-model` | `/v1/images/generations` | Image generation (per-image billing) |
| `ai-model` | `/v1/videos/generations` | Video generation (per-video billing) |
| `ai-model` | `/v1/audio/speech` | TTS and audio (per-token billing) |
| `user-api` | `/v1/uapi/{slug}/{path}` | User-published APIs (per-call billing) |
| `marketplace` | `/v1/marketplace/{service}/{path}` | Marketplace services |
| `agent-registry` | `/api/agents` | Discover other agents |

## Get Model List

The discovery endpoint doesn't include the full model list (to avoid stale data). Query models directly:

```bash
curl https://api.jarvisclaw.ai/v1/models \
  -H "Authorization: Bearer sk-your-api-key"
```

## Alternative Probe Paths

Agents commonly try these paths — all redirect to the standard endpoint:

- `/.well-known/x402/discovery` ← canonical
- `/x402/discovery` → 301 redirect
- `/discovery/resources` → 301 redirect

## OpenAPI Spec

For x402scan and automated tooling:

```bash
curl https://api.jarvisclaw.ai/openapi.json
```

Returns OpenAPI 3.1 spec with `x-payment-info` on each paid endpoint.
