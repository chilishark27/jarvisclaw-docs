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

---

## GET /price

Get an indicative price for a token swap without committing. Use this for UI display or pre-trade checks.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sellToken` | string | Yes | Token contract address to sell |
| `buyToken` | string | Yes | Token contract address to buy |
| `sellAmount` | string | Yes | Amount to sell in base units (e.g., `1000000` = 1 USDC) |
| `chainId` | integer | Yes | Target chain ID |

### Response

```json
{
  "price": "2594.12",
  "buyAmount": "2594120000",
  "sellAmount": "1000000000000000000",
  "sources": [
    { "name": "Uniswap_V3", "proportion": "0.8" },
    { "name": "Aerodrome", "proportion": "0.2" }
  ]
}
```

---

## GET /quote

Get a firm quote with transaction calldata ready for signing. Includes EIP-712 typed data for Permit2 gasless swaps.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sellToken` | string | Yes | Token contract address to sell |
| `buyToken` | string | Yes | Token contract address to buy |
| `sellAmount` | string | Yes | Amount to sell in base units |
| `chainId` | integer | Yes | Target chain ID |
| `takerAddress` | string | Yes | Taker wallet address (required for quote) |

### Response

```json
{
  "price": "2594.12",
  "buyAmount": "2594120000",
  "to": "0xdef1c0ded9bec7f1a1670819833240f027b25eff",
  "data": "0x...",
  "value": "1000000000000000000",
  "gas": "210000"
}
```

::: info Quote Expiry
Quotes are valid for **30 seconds**. After expiry, you must request a new quote.
:::

---

## POST /gasless/submit

Submit a signed gasless swap. The relayer pays gas on your behalf — the taker pays nothing beyond the swap amount.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `trade` | object | Yes | Full quote object from `/quote` response |
| `signature` | string | Yes | EIP-712 signature from taker wallet |

### Request

```json
{
  "trade": { "...full quote object..." },
  "signature": "0xabcdef1234567890..."
}
```

### Response

```json
{
  "tradeHash": "0xdef456...",
  "status": "submitted"
}
```

---

## GET /gasless/status/:tradeHash

Poll the status of a submitted gasless swap.

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tradeHash` | string | Yes | Trade hash from submit response |

### Response

```json
{
  "tradeHash": "0xdef456...",
  "status": "confirmed",
  "txHash": "0x789abc..."
}
```

Status values: `submitted`, `pending`, `confirmed`, `failed`

---

## Code Examples

::: code-group

```bash [cURL]
# Get indicative price (1 ETH -> USDC on Ethereum mainnet)
curl "https://api.jarvisclaw.ai/v1/marketplace/dex/price?sellToken=0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2&buyToken=0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48&sellAmount=1000000000000000000&chainId=1" \
  -H "Authorization: Bearer sk-your-api-key"

# Get firm quote (requires takerAddress)
curl "https://api.jarvisclaw.ai/v1/marketplace/dex/quote?sellToken=0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2&buyToken=0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48&sellAmount=1000000000000000000&chainId=1&takerAddress=0xYourWalletAddress" \
  -H "Authorization: Bearer sk-your-api-key"

# Submit gasless swap
curl -X POST https://api.jarvisclaw.ai/v1/marketplace/dex/gasless/submit \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "trade": { "...quote object..." },
    "signature": "0xYourEIP712Signature..."
  }'

# Check gasless swap status
curl "https://api.jarvisclaw.ai/v1/marketplace/dex/gasless/status/0xdef456" \
  -H "Authorization: Bearer sk-your-api-key"
```

```python [Python (API Key)]
import requests
import time

BASE = "https://api.jarvisclaw.ai/v1/marketplace/dex"
HEADERS = {"Authorization": "Bearer sk-your-api-key"}

# Token addresses on Ethereum mainnet
WETH = "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2"
USDC = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48"

# 1. Get indicative price for 1 ETH -> USDC
resp = requests.get(f"{BASE}/price", headers=HEADERS, params={
    "chainId": 1,
    "sellToken": WETH,
    "buyToken": USDC,
    "sellAmount": "1000000000000000000",  # 1 ETH (18 decimals)
})
price = resp.json()
print(f"Price: {price['price']} USDC per ETH")
print(f"Buy amount: {price['buyAmount']} (raw USDC units)")
print(f"Routed via: {[s['name'] for s in price['sources']]}")

# 2. Get firm quote
resp = requests.get(f"{BASE}/quote", headers=HEADERS, params={
    "chainId": 1,
    "sellToken": WETH,
    "buyToken": USDC,
    "sellAmount": "1000000000000000000",
    "takerAddress": "0xYourWalletAddress",
})
quote = resp.json()
print(f"Gas estimate: {quote['gas']}")

# 3. Sign the EIP-712 permit2 data (requires eth_account)
# signature = sign_eip712(quote["permit2"]["eip712"], private_key)

# 4. Submit gasless swap
resp = requests.post(f"{BASE}/gasless/submit", headers=HEADERS, json={
    "trade": quote,
    "signature": "0x<your-eip712-signature>",
})
trade_hash = resp.json()["tradeHash"]
print(f"Submitted: {trade_hash}")

# 5. Poll status
while True:
    resp = requests.get(f"{BASE}/gasless/status/{trade_hash}", headers=HEADERS)
    status = resp.json()
    if status["status"] == "confirmed":
        print(f"Swap confirmed! TX: {status['txHash']}")
        break
    elif status["status"] == "failed":
        print("Swap failed")
        break
    time.sleep(2)
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

WETH = "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2"
USDC = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48"

# Get indicative price (free — no payment triggered)
price = client.dex_price(
    chain_id=1,
    sell_token=WETH,
    buy_token=USDC,
    sell_amount="1000000000000000000",
)
print(f"Price: {price.price} USDC per ETH")
print(f"Sources: {[s.name for s in price.sources]}")

# Get firm quote
quote = client.dex_quote(
    chain_id=1,
    sell_token=WETH,
    buy_token=USDC,
    sell_amount="1000000000000000000",
    taker_address="0xYourWalletAddress",
)
print(f"Gas: {quote.gas}")

# Submit gasless swap (after signing permit2 externally)
result = client.dex_gasless_submit(trade=quote, signature="0x<signed-permit2>")
print(f"Trade hash: {result.trade_hash}")

# Poll status
status = client.dex_gasless_status(result.trade_hash)
print(f"Status: {status.status}")
if status.status == "confirmed":
    print(f"TX: {status.tx_hash}")
```

```go [Go (API Key)]
package main

import (
    "context"
    "fmt"
    "time"

    jc "github.com/api-jarvisclaw/go-sdk"
)

func main() {
    ctx := context.Background()
    mc, _ := jc.NewMarketplaceClient(jc.WithAPIKey("sk-your-api-key"))

    weth := "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2"
    usdc := "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48"

    // 1. Get indicative price
    price, _ := mc.DexPrice(ctx, &jc.DexPriceRequest{
        ChainID:    1,
        SellToken:  weth,
        BuyToken:   usdc,
        SellAmount: "1000000000000000000",
    })
    fmt.Printf("Price: %s USDC/ETH\n", price.Price)
    fmt.Printf("Buy amount: %s\n", price.BuyAmount)

    // 2. Get firm quote
    quote, _ := mc.DexQuote(ctx, &jc.DexQuoteRequest{
        ChainID:      1,
        SellToken:    weth,
        BuyToken:     usdc,
        SellAmount:   "1000000000000000000",
        TakerAddress: "0xYourWalletAddress",
    })
    fmt.Printf("Gas: %s\n", quote.Gas)

    // 3. Sign permit2 EIP-712 data and submit gasless swap
    // signature := signEIP712(quote.Permit2, privateKey)

    result, _ := mc.DexGaslessSubmit(ctx, &jc.DexGaslessSubmitRequest{
        Trade:     quote,
        Signature: "0x<your-eip712-signature>",
    })
    fmt.Printf("Trade hash: %s\n", result.TradeHash)

    // 4. Poll status
    for {
        status, _ := mc.DexGaslessStatus(ctx, result.TradeHash)
        if status.Status == "confirmed" {
            fmt.Printf("Confirmed! TX: %s\n", status.TxHash)
            break
        }
        if status.Status == "failed" {
            fmt.Println("Swap failed")
            break
        }
        time.Sleep(2 * time.Second)
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
    // DEX endpoints are free, but x402 auth still works
    mc, _ := jc.NewMarketplaceClient(jc.WithPrivateKey("0x<evm-private-key>"))

    weth := "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2"
    usdc := "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48"

    // Get price
    price, _ := mc.DexPrice(ctx, &jc.DexPriceRequest{
        ChainID:    1,
        SellToken:  weth,
        BuyToken:   usdc,
        SellAmount: "1000000000000000000",
    })
    fmt.Printf("1 ETH -> %s USDC\n", price.BuyAmount)

    // Get quote
    quote, _ := mc.DexQuote(ctx, &jc.DexQuoteRequest{
        ChainID:      1,
        SellToken:    weth,
        BuyToken:     usdc,
        SellAmount:   "1000000000000000000",
        TakerAddress: "0xYourWalletAddress",
    })

    // Sign + submit gasless swap
    result, _ := mc.DexGaslessSubmit(ctx, &jc.DexGaslessSubmitRequest{
        Trade:     quote,
        Signature: "0x<signed-permit2>",
    })
    fmt.Printf("Submitted: %s\n", result.TradeHash)

    // Check status
    status, _ := mc.DexGaslessStatus(ctx, result.TradeHash)
    fmt.Printf("Status: %s\n", status.Status)
}
```

:::

---

## Errors

| HTTP Code | Error Code | Description | Resolution |
|-----------|------------|-------------|------------|
| 400 | `insufficient_liquidity` | Not enough liquidity across available DEXes | Reduce sellAmount or try a different token pair |
| 400 | `unsupported_chain` | Chain ID not in supported list | Use supported chain: 1, 8453, 137, 42161, or 10 |
| 410 | `price_expired` | Quote expired (~30s lifetime) | Request a fresh quote |
| 400 | `invalid_token` | Token address not recognized on specified chain | Verify the contract address on the target chain |

### Error Response Format

```json
{
  "error": {
    "code": "insufficient_liquidity",
    "message": "Not enough liquidity for 1000000000 sellToken across available DEX sources on chain 8453."
  }
}
```

---

## Limitations

- **5 chains only** — Ethereum (1), Base (8453), Polygon (137), Arbitrum (42161), Optimism (10)
- **Gasless requires Permit2** — Only tokens with Permit2 approval can use gasless swaps (most ERC-20s support this)
- **30s quote expiry** — Quotes are time-sensitive; sign and submit promptly
- **No limit orders** — Only immediate market swaps; no conditional or scheduled orders
- **Price impact on large trades** — Swaps over $100k may experience significant slippage due to DEX liquidity depth
- **Token addresses required** — Use contract addresses, not symbols; verify addresses on the target chain
- **Base units only** — All amounts are in the token's smallest denomination (6 decimals for USDC, 18 for ETH/WETH)
