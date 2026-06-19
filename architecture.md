# Architecture

JarvisClaw is an AI API gateway that aggregates 40+ upstream providers with intelligent routing and on-chain micropayments via x402.

## System Overview

```
┌──────────────────────────────────────────────────────────┐
│                   JarvisClaw Gateway                      │
│                                                          │
│  ┌────────────┐  ┌─────────────┐  ┌─────────────────┐   │
│  │  API Layer │  │  AIP Layer  │  │  Marketplace    │   │
│  │            │  │  (Intent)   │  │  (Service Proxy)│   │
│  └─────┬──────┘  └──────┬──────┘  └───────┬─────────┘   │
│        │                 │                 │             │
│  ┌─────┴─────────────────┴─────────────────┴──────┐      │
│  │              Smart Router                       │      │
│  │  • Multi-model aggregation (80+ models)        │      │
│  │  • Auto failover & retry                       │      │
│  │  • Weighted load balancing                     │      │
│  └─────────────────────┬──────────────────────────┘      │
│                        │                                 │
│  ┌─────────────────────┴──────────────────────────┐      │
│  │              Payment Layer                      │      │
│  │  • x402 protocol (on-chain micropayments)      │      │
│  │  • Fiat via Airwallex                          │      │
│  │  • Token-based quota (legacy)                  │      │
│  └────────────────────────────────────────────────┘      │
└──────────────────────────────────────────────────────────┘
         │                    │                  │
         ▼                    ▼                  ▼
   ┌──────────┐      ┌──────────────┐    ┌──────────┐
   │ OpenAI   │      │ Anthropic    │    │ 40+ more │
   │ DeepSeek │      │ Google       │    │ providers│
   └──────────┘      └──────────────┘    └──────────┘
```

## Core Modules

### 1. API Layer (OpenAI-compatible)

Standard endpoints that any OpenAI SDK can call:

- `POST /v1/chat/completions` — text generation (streaming supported)
- `POST /v1/images/generations` — image generation
- `POST /v1/embeddings` — text embeddings
- `POST /v1/audio/transcriptions` — speech-to-text

Authentication: Bearer token (API key) with per-key quota and rate limits.

### 2. AIP Layer (Agent Intent Protocol)

Purpose-built for AI agents. Instead of specifying a model, agents declare **what they want to accomplish**.

- `POST /v1/intent/resolve` — declare intent, get ranked providers (free, no auth)
- `POST /v1/intent/execute` — resolve + execute in one call (x402 payment)
- `GET /v1/intent/types` — list supported intent types
- `GET /v1/providers` — list all registered providers

See [Agent Economy](/agent-economy) for details.

**Key differences from API Layer:**
- No registration required — pay per call with x402
- Intent-based routing — engine picks the best model
- Multi-dimensional scoring (cost, quality, latency, features)
- Fully isolated middleware and data path

### 3. Marketplace (Service Proxy)

Proxies third-party API services (search, prediction, etc.) with x402 payment:

```
/v1/marketplace/{service}/* → lookup config → x402 payment → forward to upstream
```

### 4. Smart Router

The routing engine that powers all layers:

- **Channel system**: each upstream connection is a "channel" with priority, weight, and model list
- **Auto model sync**: periodically detects new/deprecated models from upstreams
- **Failover**: automatic retry on different channels when one fails
- **Load balancing**: weighted random across channels of same priority

### 5. Payment Layer

Three payment methods coexist:

| Method | Use Case | Settlement |
|--------|----------|------------|
| x402 (crypto) | Agents, pay-per-call | On-chain USDC (Base/Solana) |
| Fiat (Airwallex) | Enterprise customers | Invoice / card |
| Token quota | Legacy API keys | Pre-paid balance |

## Infrastructure

| Component | Technology |
|-----------|-----------|
| Runtime | Go 1.22 + Gin |
| Database | PostgreSQL |
| Cache | Redis |
| Blockchain | Base (USDC) + Solana (USDC) |
| Facilitator | Coinbase CDP (3-node pool) |
| CDN | Cloudflare Workers + R2 |
| Load Balancer | AWS ALB |
| CI/CD | GitHub Actions → Docker → GHCR |
| Docs | VitePress → Cloudflare Pages |

## Data Flow

```
Client Request
     │
     ▼
  ALB (TLS termination)
     │
     ▼
  Gin Router ─── match path ───┐
     │                         │
     ├─ /v1/chat/* ──────► TokenAuth → Relay → Upstream
     ├─ /v1/intent/* ────► X402Seller → IntentEngine → Execute
     └─ /v1/marketplace/* ► X402Seller → ServiceProxy → Upstream
                                              │
                                              ▼
                                    Response to Client
                                              │
                                              ▼
                                    Async Settle (x402)
```

## Security

- All traffic over HTTPS (TLS 1.3)
- API keys hashed in database
- x402 payments verified via EIP-712 signature validation
- Rate limiting per IP and per key
- No credential sharing between layers (AIP ↔ API isolated)
