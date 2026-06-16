# DEX Trading API (0x Swap)

Decentralized exchange trading via 0x protocol. Best-execution routing across 100+ DEXes with Permit2 and Gasless V2. **FREE for all callers** — you only pay on-chain gas fees.

**Base URL:** `https://api.jarvisclaw.ai/v1/marketplace/dex`

## Pricing

| Endpoint | Price | Notes |
|----------|-------|-------|
| GET /price | Free | Indicative only, no commitment |
| GET /quote | Free | Firm quote with tx calldata |
| POST /gasless/submit | Free | Relayer pays gas on your behalf |
| GET /gasless/status/:tradeHash | Free | Poll swap status |

All DEX endpoints are free. No JarvisClaw fees — only standard on-chain gas costs for submitted transactions. Gasless swaps eliminate even gas fees for supported tokens.

## Supported Chains

| Chain | Chain ID | Gasless Support |
|-------|----------|-----------------|
| Ethereum | `1` | Yes |
| Base | `8453` | Yes |
| Polygon | `137` | Yes |
| Arbitrum | `42161` | Yes |
| Optimism | `10` | Yes |

## Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/price` | Indicative swap price (no commitment) |
| GET | `/quote` | Firm quote with calldata + Permit2 signature data |
| POST | `/gasless/submit` | Submit a signed gasless swap |
| GET | `/gasless/status/:tradeHash` | Track gasless swap status |
