# Prediction Markets API

Real-time prediction market data via Predexon. Access 58 endpoints across 11 categories covering Polymarket and Kalshi venues. Query markets, track wallet positions, and search across venues.

**Base URL:** `https://api.jarvisclaw.ai/v1/marketplace/prediction`

## Pricing

| Endpoint Type | Price | Description |
|---------------|-------|-------------|
| GET endpoints | $0.001/request | Market listings, details, categories |
| Wallet lookup | $0.005/request | Wallet positions and PnL queries |
| Search | $0.005/request | Cross-venue market search |

## Endpoints

### GET /polymarket/markets

List active prediction markets on Polymarket with filtering and pagination.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `limit` | integer | No | Results per page. Default: `20`, Max: `100` |
| `offset` | integer | No | Pagination offset. Default: `0` |
| `category` | string | No | Category filter: `politics`, `crypto`, `sports`, `science`, `culture`, `business` |
| `status` | string | No | Filter: `active`, `resolved`, `all`. Default: `active` |
| `sort` | string | No | Sort by: `volume`, `liquidity`, `newest`. Default: `volume` |

#### Request Example

```
GET /v1/marketplace/prediction/polymarket/markets?category=crypto&limit=5&sort=volume
```

#### Response Example

```json
{
  "markets": [
    {
      "id": "0x8f3a2c1b9e4d7f6a5b0c3e2d1a9f8b7c6d5e4f3a",
      "question": "Will BTC reach $150k by end of 2026?",
      "category": "crypto",
      "status": "active",
      "outcomes": ["Yes", "No"],
      "prices": [0.42, 0.58],
      "volume": 8750000,
      "liquidity": 1250000,
      "end_date": "2026-12-31T23:59:59Z",
      "created_at": "2025-01-15T10:00:00Z"
    },
    {
      "id": "0x1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b",
      "question": "Will ETH flip BTC market cap in 2026?",
      "category": "crypto",
      "status": "active",
      "outcomes": ["Yes", "No"],
      "prices": [0.08, 0.92],
      "volume": 3200000,
      "liquidity": 680000,
      "end_date": "2026-12-31T23:59:59Z",
      "created_at": "2025-03-20T14:30:00Z"
    }
  ],
  "total": 847,
  "limit": 5,
  "offset": 0
}
```

---

### GET /polymarket/wallet/{address}

Get wallet positions, profit/loss, and trade history for a Polymarket address.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `address` | string | Yes | Ethereum wallet address (path parameter, e.g., `0x1234...abcd`) |

#### Request Example

```
GET /v1/marketplace/prediction/polymarket/wallet/0x742d35Cc6634C0532925a3b844Bc9e7595f2bD28
```

#### Response Example

```json
{
  "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f2bD28",
  "total_pnl": 3420.50,
  "total_volume": 45000.00,
  "positions": [
    {
      "market_id": "0x8f3a2c1b9e4d7f6a5b0c3e2d1a9f8b7c6d5e4f3a",
      "question": "Will BTC reach $150k by end of 2026?",
      "outcome": "Yes",
      "shares": 1200,
      "avg_price": 0.35,
      "current_price": 0.42,
      "unrealized_pnl": 84.00
    },
    {
      "market_id": "0x1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b",
      "question": "Will ETH flip BTC market cap in 2026?",
      "outcome": "No",
      "shares": 500,
      "avg_price": 0.88,
      "current_price": 0.92,
      "unrealized_pnl": 20.00
    }
  ],
  "resolved_pnl": 3316.50,
  "open_positions": 2
}
```

---

### GET /kalshi/markets

List prediction markets on Kalshi with filtering and pagination.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `limit` | integer | No | Results per page. Default: `20`, Max: `100` |
| `offset` | integer | No | Pagination offset. Default: `0` |
| `category` | string | No | Category filter: `politics`, `economics`, `tech`, `climate`, `finance` |
| `status` | string | No | Filter: `open`, `closed`, `settled`. Default: `open` |
| `series_ticker` | string | No | Filter by series (e.g., `KXBTC`, `KXINX`) |
| `cursor` | string | No | Pagination cursor (alternative to offset) |

#### Request Example

```
GET /v1/marketplace/prediction/kalshi/markets?category=economics&status=open&limit=5
```

#### Response Example

```json
{
  "markets": [
    {
      "ticker": "KXBTC-26DEC31-T150000",
      "title": "Bitcoin above $150,000 on December 31?",
      "status": "open",
      "category": "finance",
      "yes_price": 42,
      "no_price": 58,
      "volume": 285000,
      "open_interest": 120000,
      "close_time": "2026-12-31T23:59:59Z"
    },
    {
      "ticker": "KXINX-26JUN30-T6000",
      "title": "S&P 500 above 6000 on June 30?",
      "status": "open",
      "category": "finance",
      "yes_price": 71,
      "no_price": 29,
      "volume": 450000,
      "open_interest": 210000,
      "close_time": "2026-06-30T20:00:00Z"
    }
  ],
  "cursor": "eyJsYXN0X2lkIjoiS1hJTlgtMjZKVU4zMC1UNjAwMCJ9",
  "total": 312
}
```

---

### GET /markets/search

Search across all supported prediction market venues simultaneously.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | Yes | Search query (e.g., "bitcoin", "election", "AI") |
| `providers` | string | No | Comma-separated venue filter: `polymarket`, `kalshi`. Default: all |
| `limit` | integer | No | Max results. Default: `20`, Max: `50` |
| `status` | string | No | Market status filter |

#### Request Example

```
GET /v1/marketplace/prediction/markets/search?q=bitcoin%20150k&providers=polymarket,kalshi&limit=10
```

#### Response Example

```json
{
  "results": [
    {
      "venue": "polymarket",
      "id": "0x8f3a2c1b9e4d7f6a5b0c3e2d1a9f8b7c6d5e4f3a",
      "question": "Will BTC reach $150k by end of 2026?",
      "yes_price": 0.42,
      "volume": 8750000,
      "status": "active",
      "end_date": "2026-12-31T23:59:59Z"
    },
    {
      "venue": "kalshi",
      "id": "KXBTC-26DEC31-T150000",
      "question": "Bitcoin above $150,000 on December 31?",
      "yes_price": 0.42,
      "volume": 285000,
      "status": "open",
      "end_date": "2026-12-31T23:59:59Z"
    }
  ],
  "total": 2,
  "query": "bitcoin 150k"
}
```

## Errors

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 400 | `invalid_address` | Wallet address is not a valid Ethereum address |
| 404 | `market_not_found` | No market exists with the specified ID or ticker |
| 404 | `market_closed` | Market has been resolved and is no longer queryable for live data |

### Error Response Format

```json
{
  "error": {
    "code": "invalid_address",
    "message": "The address 'abc123' is not a valid Ethereum address. Expected 0x-prefixed 40-character hex string."
  }
}
```

## Code Examples

::: code-group

```bash [cURL]
# List active Polymarket crypto markets
curl "https://api.jarvisclaw.ai/v1/marketplace/prediction/polymarket/markets?category=crypto&limit=5&sort=volume" \
  -H "Authorization: Bearer sk-your-api-key"

# Get wallet positions and PnL
curl "https://api.jarvisclaw.ai/v1/marketplace/prediction/polymarket/wallet/0x742d35Cc6634C0532925a3b844Bc9e7595f2bD28" \
  -H "Authorization: Bearer sk-your-api-key"

# List Kalshi markets
curl "https://api.jarvisclaw.ai/v1/marketplace/prediction/kalshi/markets?status=open&limit=10" \
  -H "Authorization: Bearer sk-your-api-key"

# Cross-venue search
curl "https://api.jarvisclaw.ai/v1/marketplace/prediction/markets/search?q=bitcoin%20150k&providers=polymarket,kalshi" \
  -H "Authorization: Bearer sk-your-api-key"
```

```python [Python (API Key)]
import requests

BASE = "https://api.jarvisclaw.ai/v1/marketplace/prediction"
HEADERS = {
    "Authorization": "Bearer sk-your-api-key",
}

# List active Polymarket markets by volume
resp = requests.get(f"{BASE}/polymarket/markets", headers=HEADERS, params={
    "category": "crypto",
    "sort": "volume",
    "limit": 10,
})
markets = resp.json()
for market in markets["markets"]:
    yes_price = market["prices"][0]
    print(f"{market['question']} — Yes: {yes_price:.0%} (vol: ${market['volume']:,.0f})")

# Check wallet positions
resp = requests.get(
    f"{BASE}/polymarket/wallet/0x742d35Cc6634C0532925a3b844Bc9e7595f2bD28",
    headers=HEADERS,
)
wallet = resp.json()
print(f"\nWallet PnL: ${wallet['total_pnl']:,.2f}")
for pos in wallet["positions"]:
    print(f"  {pos['question']}: {pos['outcome']} @ {pos['current_price']:.2f} (PnL: ${pos['unrealized_pnl']:.2f})")

# List Kalshi markets
resp = requests.get(f"{BASE}/kalshi/markets", headers=HEADERS, params={
    "status": "open",
    "category": "finance",
    "limit": 5,
})
for market in resp.json()["markets"]:
    print(f"{market['title']} — Yes: {market['yes_price']}c")

# Cross-venue search
resp = requests.get(f"{BASE}/markets/search", headers=HEADERS, params={
    "q": "bitcoin 150k",
    "providers": "polymarket,kalshi",
})
for result in resp.json()["results"]:
    print(f"[{result['venue']}] {result['question']} — Yes: {result['yes_price']}")
```

```python [Python (x402 Agent)]
from jarvisclaw import PredictionClient

# x402 Agent wallet — pays per-call via USDC
# Base chain (EVM)
pred = PredictionClient(private_key="0x<evm-private-key>")

# Or Solana
# pred = PredictionClient(private_key="<solana-bs58-keypair>")

# List active Polymarket markets
markets = pred.polymarket_markets(category="crypto", sort="volume", limit=10)
for market in markets:
    print(f"{market.question} — Yes: {market.prices[0]:.0%}")

# Check wallet positions
wallet = pred.polymarket_wallet("0x742d35Cc6634C0532925a3b844Bc9e7595f2bD28")
print(f"Total PnL: ${wallet.total_pnl:,.2f}")
for pos in wallet.positions:
    print(f"  {pos.question}: {pos.outcome} (PnL: ${pos.unrealized_pnl:.2f})")

# List Kalshi markets
kalshi = pred.kalshi_markets(status="open", category="finance", limit=5)
for market in kalshi:
    print(f"{market.title} — Yes: {market.yes_price}c")

# Cross-venue search
results = pred.search("bitcoin 150k", providers=["polymarket", "kalshi"])
for r in results:
    print(f"[{r.venue}] {r.question} — Yes: {r.yes_price}")
```

```go [Go (API Key)]
package main

import (
    "context"
    "fmt"
    jc "github.com/api-jarvisclaw/go-sdk"
)

func main() {
    ctx := context.Background()
    pred, _ := jc.NewPredictionClient(jc.WithAPIKey("sk-your-api-key"))

    // List active Polymarket markets
    markets, _ := pred.PolymarketMarkets(ctx, &jc.MarketsQuery{
        Category: "crypto",
        Sort:     "volume",
        Limit:    10,
    })
    for _, m := range markets.Markets {
        fmt.Printf("%s — Yes: %.0f%%\n", m.Question, m.Prices[0]*100)
    }

    // Check wallet positions
    wallet, _ := pred.PolymarketWallet(ctx, "0x742d35Cc6634C0532925a3b844Bc9e7595f2bD28")
    fmt.Printf("Total PnL: $%.2f\n", wallet.TotalPnL)
    for _, pos := range wallet.Positions {
        fmt.Printf("  %s: %s (PnL: $%.2f)\n", pos.Question, pos.Outcome, pos.UnrealizedPnL)
    }

    // List Kalshi markets
    kalshi, _ := pred.KalshiMarkets(ctx, &jc.KalshiQuery{
        Status:   "open",
        Category: "finance",
        Limit:    5,
    })
    for _, m := range kalshi.Markets {
        fmt.Printf("%s — Yes: %dc\n", m.Title, m.YesPrice)
    }

    // Cross-venue search
    results, _ := pred.Search(ctx, &jc.SearchQuery{
        Q:         "bitcoin 150k",
        Providers: []string{"polymarket", "kalshi"},
    })
    for _, r := range results.Results {
        fmt.Printf("[%s] %s — Yes: %.2f\n", r.Venue, r.Question, r.YesPrice)
    }
}
```

```go [Go (x402 Agent)]
package main

import (
    "context"
    "fmt"
    jc "github.com/api-jarvisclaw/go-sdk"
)

func main() {
    ctx := context.Background()

    // x402 Agent wallet — pays per-call via USDC on Base
    pred, _ := jc.NewPredictionClient(jc.WithPrivateKey("0x<evm-private-key>"))

    // List active Polymarket markets
    markets, _ := pred.PolymarketMarkets(ctx, &jc.MarketsQuery{
        Category: "crypto",
        Sort:     "volume",
        Limit:    10,
    })
    for _, m := range markets.Markets {
        fmt.Printf("%s — Yes: %.0f%%\n", m.Question, m.Prices[0]*100)
    }

    // Check wallet positions
    wallet, _ := pred.PolymarketWallet(ctx, "0x742d35Cc6634C0532925a3b844Bc9e7595f2bD28")
    fmt.Printf("Total PnL: $%.2f\n", wallet.TotalPnL)

    // Cross-venue search
    results, _ := pred.Search(ctx, &jc.SearchQuery{
        Q:         "bitcoin 150k",
        Providers: []string{"polymarket", "kalshi"},
    })
    for _, r := range results.Results {
        fmt.Printf("[%s] %s — Yes: %.2f\n", r.Venue, r.Question, r.YesPrice)
    }
}
```

:::

## Limitations

- **Read-only** — order placement, position management, and trading are not supported
- **No WebSocket** — real-time streaming is not available; poll endpoints for updates (data refreshes every ~5s)
- **Ethereum addresses only** — wallet lookups require full `0x`-prefixed Ethereum addresses, ENS names are not resolved
- **Sports markets delayed** — sports category data may lag by up to 60 seconds due to upstream rate limits
- **Cross-venue matching is best-effort** — the search endpoint matches markets across venues by question similarity, not guaranteed 1:1 pairing
- **Price format differs by venue** — Polymarket uses decimals (0.00-1.00), Kalshi uses cents (0-100)
- **Rate limit** — 100 requests/minute per API key
