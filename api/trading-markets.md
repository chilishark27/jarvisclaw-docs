# Trading Markets API

Real-time price data for traditional markets. 1,746+ equities across 12 global exchanges with ~400ms oracle cadence. Plus 500+ crypto pairs, 30+ forex pairs, and commodities. Stocks $0.001/call; crypto/FX/commodities free.

**Base URL:** `https://api.jarvisclaw.ai/v1/marketplace/markets`

## Pricing

| Asset Class | Price | Coverage |
|-------------|-------|----------|
| Stocks | $0.001/request | 1,746+ tickers across 12 exchanges |
| Crypto | Free | 500+ trading pairs |
| Forex | Free | 30+ currency pairs |
| Commodities | Free | Gold, silver, oil, natural gas, and more |

## Endpoints

| Method | Endpoint | Description | Price |
|--------|----------|-------------|-------|
| GET | `/stocks/:market/price/:symbol` | Stock price | $0.001 |
| GET | `/crypto/price/:pair` | Crypto price | Free |
| GET | `/fx/price/:pair` | Forex rate | Free |
| GET | `/commodity/price/:symbol` | Commodity price | Free |

## Supported Markets

| Code | Exchange | Example Symbols |
|------|----------|-----------------|
| `us` | US (NYSE/NASDAQ) | AAPL, TSLA, NVDA |
| `jp` | Tokyo Stock Exchange | 7203, 6758, 9984 |
| `uk` | London Stock Exchange | SHEL, AZN, HSBA |
| `kr` | Korea Exchange | 005930, 000660 |
| `hk` | Hong Kong Exchange | 0700, 9988, 1299 |
| `de` | Deutsche Boerse (XETRA) | SAP, SIE, ALV |
| `fr` | Euronext Paris | MC, OR, AI |
| `nl` | Euronext Amsterdam | ASML, INGA, UNA |
| `ie` | Euronext Dublin | CRH, RYA |
| `lu` | Luxembourg Stock Exchange | ARCE |
| `cn` | Shanghai/Shenzhen | 600519, 000858 |
| `ca` | Toronto Stock Exchange | RY, TD, SHOP |

---

## GET /stocks/:market/price/:symbol

Real-time stock price from one of 12 supported global exchanges.

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `market` | string | Yes | Exchange code: `us`, `jp`, `uk`, `kr`, `hk`, `de`, `fr`, `nl`, `ie`, `lu`, `cn`, `ca` |
| `symbol` | string | Yes | Ticker symbol (e.g., `AAPL`, `7203`, `ASML`) |

### Response

```json
{
  "symbol": "AAPL",
  "market": "us",
  "price": 301.64,
  "change": 3.25,
  "change_percent": 1.09,
  "volume": 52000000,
  "timestamp": 1717200000
}
```

---

## GET /crypto/price/:pair

Real-time cryptocurrency price. **Free** — no charges.

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `pair` | string | Yes | Trading pair with dash separator (e.g., `BTC-USD`, `ETH-USD`, `SOL-USD`) |

### Response

```json
{
  "pair": "BTC-USD",
  "price": 63895.25,
  "change_24h": 2.35,
  "high_24h": 64200.00,
  "low_24h": 62800.00,
  "volume_24h": 25000000000,
  "timestamp": 1717200000
}
```

---

## GET /fx/price/:pair

Real-time forex exchange rate. **Free** — no charges.

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `pair` | string | Yes | Currency pair with dash separator (e.g., `EUR-USD`, `GBP-JPY`, `USD-JPY`) |

### Response

```json
{
  "pair": "EUR-USD",
  "price": 1.1538,
  "change_24h": -0.15,
  "high_24h": 1.1560,
  "low_24h": 1.1510,
  "timestamp": 1717200000
}
```

---

## GET /commodity/price/:symbol

Real-time commodity price. **Free** — no charges.

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `symbol` | string | Yes | Commodity symbol (e.g., `XAU-USD`, `XAG-USD`, `WTI-USD`, `NG-USD`) |

### Response

```json
{
  "symbol": "XAU-USD",
  "price": 4337.30,
  "unit": "USD/oz",
  "change_24h": 0.85,
  "timestamp": 1717200000
}
```

---

## Code Examples

::: code-group

```bash [cURL]
# US stock price (Apple)
curl "https://api.jarvisclaw.ai/v1/marketplace/markets/stocks/us/price/AAPL" \
  -H "Authorization: Bearer sk-your-api-key"

# Japanese stock price (Toyota - ticker 7203)
curl "https://api.jarvisclaw.ai/v1/marketplace/markets/stocks/jp/price/7203" \
  -H "Authorization: Bearer sk-your-api-key"

# Crypto price (free)
curl "https://api.jarvisclaw.ai/v1/marketplace/markets/crypto/price/BTC-USD" \
  -H "Authorization: Bearer sk-your-api-key"

# Forex rate (free)
curl "https://api.jarvisclaw.ai/v1/marketplace/markets/fx/price/EUR-USD" \
  -H "Authorization: Bearer sk-your-api-key"

# Commodity price (free)
curl "https://api.jarvisclaw.ai/v1/marketplace/markets/commodity/price/XAU-USD" \
  -H "Authorization: Bearer sk-your-api-key"
```

```python [Python (API Key)]
import requests

BASE = "https://api.jarvisclaw.ai/v1/marketplace/markets"
HEADERS = {"Authorization": "Bearer sk-your-api-key"}

# US stock price
resp = requests.get(f"{BASE}/stocks/us/price/AAPL", headers=HEADERS)
aapl = resp.json()
print(f"AAPL: ${aapl['price']} ({aapl['change_percent']:+.2f}%)")

# Japanese stock (Toyota)
resp = requests.get(f"{BASE}/stocks/jp/price/7203", headers=HEADERS)
toyota = resp.json()
print(f"Toyota (7203): \u00a5{toyota['price']}")

# Crypto price (free)
resp = requests.get(f"{BASE}/crypto/price/BTC-USD", headers=HEADERS)
btc = resp.json()
print(f"BTC: ${btc['price']:,.2f} ({btc['change_24h']:+.2f}%)")
print(f"  24h range: ${btc['low_24h']:,.2f} - ${btc['high_24h']:,.2f}")

# Forex rate (free)
resp = requests.get(f"{BASE}/fx/price/EUR-USD", headers=HEADERS)
eur = resp.json()
print(f"EUR/USD: {eur['price']}")

# Commodity price (free)
resp = requests.get(f"{BASE}/commodity/price/XAU-USD", headers=HEADERS)
gold = resp.json()
print(f"Gold: ${gold['price']}/{gold['unit'].split('/')[1]}")
```

```python [Python (x402 Agent)]
from jarvisclaw import MarketplaceClient

# --- Option A: Base chain (EVM) ---
# Hex private key -> USDC on Base (Chain ID 8453)
client = MarketplaceClient(private_key="0x<evm-private-key>")

# --- Option B: Solana ---
# Base58 keypair -> USDC SPL on Solana mainnet
# client = MarketplaceClient(private_key="<solana-bs58-keypair>")

# SDK auto-detects chain from key format - no config needed

# US stock price ($0.001 — auto-paid via x402)
aapl = client.stock_price(market="us", symbol="AAPL")
print(f"AAPL: ${aapl.price} ({aapl.change_percent:+.2f}%)")

# Japanese stock
toyota = client.stock_price(market="jp", symbol="7203")
print(f"Toyota: \u00a5{toyota.price}")

# Crypto price (free — no payment triggered)
btc = client.crypto_price("BTC-USD")
print(f"BTC: ${btc.price:,.2f} (24h: {btc.change_24h:+.2f}%)")

# Forex rate (free)
eur = client.fx_price("EUR-USD")
print(f"EUR/USD: {eur.price}")

# Commodity price (free)
gold = client.commodity_price("XAU-USD")
print(f"Gold: ${gold.price}/{gold.unit}")
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
    mc, _ := jc.NewMarketplaceClient(jc.WithAPIKey("sk-your-api-key"))

    // US stock price
    aapl, _ := mc.StockPrice(ctx, "us", "AAPL")
    fmt.Printf("AAPL: $%.2f (%+.2f%%)\n", aapl.Price, aapl.ChangePercent)

    // Japanese stock (Toyota)
    toyota, _ := mc.StockPrice(ctx, "jp", "7203")
    fmt.Printf("Toyota: %.0f\n", toyota.Price)

    // Crypto price (free)
    btc, _ := mc.CryptoPrice(ctx, "BTC-USD")
    fmt.Printf("BTC: $%.2f (24h: %+.2f%%)\n", btc.Price, btc.Change24h)
    fmt.Printf("  Range: $%.2f - $%.2f\n", btc.Low24h, btc.High24h)

    // Forex rate (free)
    eur, _ := mc.FxPrice(ctx, "EUR-USD")
    fmt.Printf("EUR/USD: %.4f\n", eur.Price)

    // Commodity price (free)
    gold, _ := mc.CommodityPrice(ctx, "XAU-USD")
    fmt.Printf("Gold: $%.2f/%s\n", gold.Price, gold.Unit)
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
    mc, _ := jc.NewMarketplaceClient(jc.WithPrivateKey("0x<evm-private-key>"))

    // US stock price ($0.001 — auto-paid via x402)
    aapl, _ := mc.StockPrice(ctx, "us", "AAPL")
    fmt.Printf("AAPL: $%.2f (%+.2f%%)\n", aapl.Price, aapl.ChangePercent)

    // Japanese stock
    toyota, _ := mc.StockPrice(ctx, "jp", "7203")
    fmt.Printf("Toyota: %.0f\n", toyota.Price)

    // Crypto price (free — no payment triggered)
    btc, _ := mc.CryptoPrice(ctx, "BTC-USD")
    fmt.Printf("BTC: $%.2f\n", btc.Price)

    // Forex rate (free)
    eur, _ := mc.FxPrice(ctx, "EUR-USD")
    fmt.Printf("EUR/USD: %.4f\n", eur.Price)

    // Commodity price (free)
    gold, _ := mc.CommodityPrice(ctx, "XAU-USD")
    fmt.Printf("Gold: $%.2f/%s\n", gold.Price, gold.Unit)
}
```

:::

---

## Errors

| HTTP Code | Error Code | Description | Resolution |
|-----------|------------|-------------|------------|
| 404 | `symbol_not_found` | Ticker does not exist in specified market | Check symbol spelling and market code |
| 404 | `market_not_supported` | Not one of 12 supported exchanges | Use: us, jp, uk, kr, hk, de, fr, nl, ie, lu, cn, ca |
| 503 | `market_closed` | Exchange currently closed | Returns last known price with stale timestamp |

### Error Response Format

```json
{
  "error": {
    "code": "symbol_not_found",
    "message": "Ticker 'XYZ' not found in market 'us'. Verify the symbol exists on this exchange."
  }
}
```

---

## Limitations

- **Stock prices only during trading hours** — Returns last known price when exchange is closed (check timestamp)
- **No historical OHLC data** — Only real-time snapshots; use Surf crypto-data API for candlestick history
- **12 supported markets only** — No OTC, pink sheets, or emerging market exchanges
- **~400ms cadence** — Not suitable for high-frequency trading; designed for display and agent decision-making
- **Read-only** — No order placement; use DEX Trading API for on-chain crypto trades
- **Trading pair format** — Use dash separator (e.g., `BTC-USD`, not `BTC/USD` or `BTCUSD`)
- **All prices in USD** — Unless otherwise specified in the response `unit` field
