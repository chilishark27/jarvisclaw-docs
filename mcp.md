# MCP Configuration

JarvisClaw exposes an MCP (Model Context Protocol) server. Connect from Claude Code, Cursor, Windsurf, or any MCP client to use AI models as tools.

## Authentication

| Method | Header | Use case |
|--------|--------|----------|
| API Key | `Authorization: Bearer sk-...` | Pre-paid account |
| x402 USDC | `PAYMENT-SIGNATURE: ...` | No account, wallet direct pay |

Discovery methods (`initialize`, `tools/list`, `resources/list`) are free — no auth needed. Only `tools/call` requires payment.

## Claude Code

Add to `~/.claude.json`:

```json
{
  "mcpServers": {
    "jarvisclaw": {
      "type": "url",
      "url": "https://api.jarvisclaw.ai/mcp",
      "headers": {
        "Authorization": "Bearer sk-your-api-key"
      }
    }
  }
}
```

After config, these tools appear automatically:

| Tool | Description |
|------|-------------|
| `list_models` | List all available AI models |
| `chat` | Call any model (OpenAI-compatible) |
| `search_apis` | Search user-published APIs |
| `get_api_detail` | Get API endpoint details |
| `discover_agents` | Find other registered agents |
| `uapi_{slug}` | Call a specific user API directly |

## Cursor / Windsurf

Add to `.cursor/mcp.json` in your project root:

```json
{
  "mcpServers": {
    "jarvisclaw": {
      "url": "https://api.jarvisclaw.ai/mcp",
      "headers": {
        "Authorization": "Bearer sk-your-api-key"
      }
    }
  }
}
```

## Programmatic Usage

MCP uses JSON-RPC 2.0 over HTTP. Connect directly:

```python
import httpx

# 1. Initialize (no auth needed)
resp = httpx.post("https://api.jarvisclaw.ai/mcp", json={
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
        "protocolVersion": "2025-03-26",
        "capabilities": {},
        "clientInfo": {"name": "my-agent", "version": "1.0"}
    }
})

# 2. List tools (no auth needed)
resp = httpx.post("https://api.jarvisclaw.ai/mcp", json={
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/list"
})

# 3. Call a tool (auth required)
resp = httpx.post("https://api.jarvisclaw.ai/mcp",
    headers={"Authorization": "Bearer sk-your-api-key"},
    json={
        "jsonrpc": "2.0",
        "id": 3,
        "method": "tools/call",
        "params": {
            "name": "chat",
            "arguments": {
                "model": "gpt-4o",
                "messages": [{"role": "user", "content": "Hello!"}]
            }
        }
    }
)
print(resp.json()["result"]["content"][0]["text"])
```

## x402 Payment via MCP

Agents with a USDC wallet can pay per call without an account:

```
Agent                        JarvisClaw MCP
  │                                │
  │── tools/call (no auth) ──────> │
  │<── error: payment required ──  │  (price, payTo, network)
  │                                │
  │  [sign USDC x402 payment]      │
  │                                │
  │── tools/call + PAYMENT-SIG ──> │
  │<── tool result ──────────────  │
```

```python
# Call without payment → get price info
resp = httpx.post("https://api.jarvisclaw.ai/mcp", json={
    "jsonrpc": "2.0", "id": 3,
    "method": "tools/call",
    "params": {"name": "chat", "arguments": {"model": "gpt-4o", "messages": [...]}}
})
# → error with payment details (amount, payTo, network)

# Sign x402 payment, then retry with PAYMENT-SIGNATURE header
resp = httpx.post("https://api.jarvisclaw.ai/mcp",
    headers={"PAYMENT-SIGNATURE": payment_sig},
    json={...}
)
```

Payment parameters:
- **Network:** Base (eip155:8453)
- **Asset:** USDC (`0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`)
- **Facilitator:** `https://api.cdp.coinbase.com/platform/v2/x402`
