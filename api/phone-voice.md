# Phone & Voice API

AI-powered outbound voice calls (Bland.ai) and wallet-owned phone numbers (Twilio). Make conversational AI calls with real-time transcripts, buy/manage dedicated US/CA numbers, and perform carrier/fraud lookups. Outbound only, no SMS.

**Base URL:** `https://api.jarvisclaw.ai/v1/marketplace/phone`

## Pricing

| Endpoint | Price | Description |
|----------|-------|-------------|
| POST `/voice/call` | $0.54/call | Initiate AI voice call |
| GET `/voice/call/:call_id` | Free | Retrieve call status/transcript |
| POST `/lookup` | $0.01/request | Carrier identification |
| POST `/lookup/fraud` | $0.05/request | Fraud risk assessment |
| POST `/numbers/buy` | $5.00/number | Lease a phone number (30-day) |
| POST `/numbers/list` | $0.001/request | List owned numbers |

## Endpoints

### POST /voice/call

Initiate an outbound AI voice call. The AI agent handles the conversation using the provided task/system prompt. Powered by Bland.ai.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `to` | string | Yes | Destination phone number (E.164 format, e.g., `+14155551234`) |
| `from` | string | No | Caller ID — must be a leased number from `/numbers/buy` |
| `task` | string | Yes | Instructions for the AI voice agent (system prompt) |
| `voice` | string | No | Voice preset. Default: `nova` |
| `max_duration` | integer | No | Max call duration in seconds. Default: `1800` (30 min) |
| `webhook_url` | string | No | URL for real-time call status events |
| `first_sentence` | string | No | Override the AI's opening line |
| `wait_for_greeting` | boolean | No | Wait for callee to speak first. Default: `true` |

#### Request Example

```json
{
  "to": "+14155551234",
  "from": "+14155559876",
  "task": "You are an appointment reminder assistant. Confirm the user's dental appointment for Tuesday June 17 at 2 PM. Be polite and concise.",
  "voice": "nova",
  "max_duration": 120,
  "first_sentence": "Hi, this is a reminder call from Dr. Smith's office.",
  "wait_for_greeting": true
}
```

#### Response Example

```json
{
  "call_id": "call_abc123def456",
  "status": "initiated",
  "to": "+14155551234",
  "from": "+14155559876",
  "created_at": "2025-06-13T12:00:00Z"
}
```

---

### GET /voice/call/:call_id

Retrieve the full transcript, status, and metadata for a call.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `call_id` | string | Yes | Call ID (path parameter) |

#### Response Example

```json
{
  "call_id": "call_abc123def456",
  "status": "completed",
  "duration_seconds": 45,
  "to": "+14155551234",
  "from": "+14155559876",
  "transcript": [
    {"role": "assistant", "content": "Hi, this is a reminder call from Dr. Smith's office."},
    {"role": "user", "content": "Yes, hello?"},
    {"role": "assistant", "content": "I'm calling to confirm your dental appointment for Tuesday June 17 at 2 PM. Can you confirm you'll be there?"},
    {"role": "user", "content": "Yes, I'll be there. Thank you."},
    {"role": "assistant", "content": "Great, you're all set. Have a wonderful day!"}
  ],
  "summary": "User confirmed dental appointment for Tuesday June 17 at 2 PM.",
  "created_at": "2025-06-13T12:00:00Z",
  "ended_at": "2025-06-13T12:00:45Z"
}
```

Call statuses: `initiated`, `ringing`, `in_progress`, `completed`, `failed`, `no_answer`, `busy`

---

### POST /lookup

Identify the carrier and line type for a phone number.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `phoneNumber` | string | Yes | Phone number in E.164 format |

#### Request Example

```json
{
  "phoneNumber": "+14155551234"
}
```

#### Response Example

```json
{
  "phoneNumber": "+14155551234",
  "carrier": "T-Mobile",
  "line_type": "mobile",
  "country": "US",
  "country_code": "1"
}
```

---

### POST /lookup/fraud

Assess fraud risk and identify suspicious indicators for a phone number.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `phoneNumber` | string | Yes | Phone number in E.164 format |

#### Request Example

```json
{
  "phoneNumber": "+14155551234"
}
```

#### Response Example

```json
{
  "phoneNumber": "+14155551234",
  "risk_score": 15,
  "risk_level": "low",
  "flags": [],
  "carrier": "T-Mobile",
  "line_type": "mobile",
  "is_voip": false,
  "is_prepaid": false
}
```

Risk levels: `low` (0-25), `medium` (26-50), `high` (51-75), `critical` (76-100)

---

### POST /numbers/buy

Lease a dedicated phone number for use as a caller ID on outbound calls.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `country` | string | Yes | ISO country code (`US` or `CA`) |
| `area_code` | string | No | Preferred area code (e.g., `415`) |

#### Request Example

```json
{
  "country": "US",
  "area_code": "415"
}
```

#### Response Example

```json
{
  "number": "+14155559876",
  "country": "US",
  "capabilities": ["voice"],
  "monthly_cost": 5.00,
  "lease_start": "2025-06-13T12:00:00Z",
  "expires_at": "2025-07-13T12:00:00Z"
}
```

---

### POST /numbers/list

List all phone numbers currently leased to your account.

#### Request Example

```json
{}
```

#### Response Example

```json
{
  "numbers": [
    {
      "number": "+14155559876",
      "country": "US",
      "capabilities": ["voice"],
      "monthly_cost": 5.00,
      "lease_start": "2025-06-01T12:00:00Z",
      "expires_at": "2025-07-01T12:00:00Z",
      "status": "active"
    }
  ],
  "total": 1
}
```

## Errors

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 400 | `invalid_phone_number` | Phone number is not valid E.164 format |
| 404 | `number_not_owned` | The `from` number is not leased to your account |
| 404 | `call_not_found` | No call found with the specified `call_id` |
| 409 | `number_unavailable` | Requested number or area code has no availability |
| 500 | `call_failed` | Voice call could not be connected (carrier/network issue) |

### Error Response Format

```json
{
  "error": {
    "code": "invalid_phone_number",
    "message": "The phone number '+1415555' is not valid E.164 format. Expected format: +14155551234"
  }
}
```

## Code Examples

::: code-group

```bash [cURL]
# Initiate a voice call
curl -X POST https://api.jarvisclaw.ai/v1/marketplace/phone/voice/call \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "+14155551234",
    "task": "You are a friendly appointment reminder. Confirm the dental appointment for Tuesday at 2 PM.",
    "voice": "nova",
    "max_duration": 120
  }'

# Get call transcript
curl https://api.jarvisclaw.ai/v1/marketplace/phone/voice/call/call_abc123def456 \
  -H "Authorization: Bearer sk-your-api-key"

# Carrier lookup
curl -X POST https://api.jarvisclaw.ai/v1/marketplace/phone/lookup \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"phoneNumber": "+14155551234"}'

# Fraud risk check
curl -X POST https://api.jarvisclaw.ai/v1/marketplace/phone/lookup/fraud \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"phoneNumber": "+14155551234"}'

# Buy a phone number
curl -X POST https://api.jarvisclaw.ai/v1/marketplace/phone/numbers/buy \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"country": "US", "area_code": "415"}'

# List owned numbers
curl -X POST https://api.jarvisclaw.ai/v1/marketplace/phone/numbers/list \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{}'
```

```python [Python (API Key)]
import requests

BASE = "https://api.jarvisclaw.ai/v1/marketplace/phone"
HEADERS = {
    "Authorization": "Bearer sk-your-api-key",
    "Content-Type": "application/json",
}

# Initiate an AI voice call
resp = requests.post(f"{BASE}/voice/call", headers=HEADERS, json={
    "to": "+14155551234",
    "task": "You are an appointment reminder assistant. Confirm the dental appointment for Tuesday at 2 PM.",
    "voice": "nova",
    "max_duration": 120,
})
call = resp.json()
print(f"Call initiated: {call['call_id']}")

# Get transcript after call completes
resp = requests.get(
    f"{BASE}/voice/call/{call['call_id']}",
    headers={"Authorization": "Bearer sk-your-api-key"},
)
transcript = resp.json()
print(f"Status: {transcript['status']}, Duration: {transcript['duration_seconds']}s")
for turn in transcript["transcript"]:
    print(f"  {turn['role']}: {turn['content']}")

# Carrier lookup
resp = requests.post(f"{BASE}/lookup", headers=HEADERS, json={
    "phoneNumber": "+14155551234",
})
info = resp.json()
print(f"Carrier: {info['carrier']}, Type: {info['line_type']}")

# Fraud risk assessment
resp = requests.post(f"{BASE}/lookup/fraud", headers=HEADERS, json={
    "phoneNumber": "+14155551234",
})
risk = resp.json()
print(f"Risk: {risk['risk_level']} (score: {risk['risk_score']})")

# Buy a number
resp = requests.post(f"{BASE}/numbers/buy", headers=HEADERS, json={
    "country": "US",
    "area_code": "415",
})
print(f"Leased: {resp.json()['number']}")

# List owned numbers
resp = requests.post(f"{BASE}/numbers/list", headers=HEADERS, json={})
for num in resp.json()["numbers"]:
    print(f"  {num['number']} — expires {num['expires_at']}")
```

```python [Python (x402 Agent)]
from jarvisclaw import MarketplaceClient

# x402 Agent wallet — pays per-call via USDC
# Base chain (EVM)
client = MarketplaceClient(private_key="0x<evm-private-key>")

# Or Solana
# client = MarketplaceClient(private_key="<solana-bs58-keypair>")

# Initiate an AI voice call
call = client.call("phone", "/voice/call", method="POST", json={
    "to": "+14155551234",
    "task": "You are an appointment reminder assistant. Confirm the dental appointment for Tuesday at 2 PM.",
    "voice": "nova",
    "max_duration": 120,
})
print(f"Call initiated: {call['call_id']}")

# Get transcript
transcript = client.call("phone", f"/voice/call/{call['call_id']}")
print(f"Status: {transcript['status']}, Duration: {transcript['duration_seconds']}s")
for turn in transcript["transcript"]:
    print(f"  {turn['role']}: {turn['content']}")

# Carrier lookup
info = client.call("phone", "/lookup", method="POST", json={
    "phoneNumber": "+14155551234",
})
print(f"Carrier: {info['carrier']}, Type: {info['line_type']}")

# Fraud risk assessment
risk = client.call("phone", "/lookup/fraud", method="POST", json={
    "phoneNumber": "+14155551234",
})
print(f"Risk: {risk['risk_level']} (score: {risk['risk_score']})")

# Buy a number
number = client.call("phone", "/numbers/buy", method="POST", json={
    "country": "US",
    "area_code": "415",
})
print(f"Leased: {number['number']}")

# List owned numbers
numbers = client.call("phone", "/numbers/list", method="POST", json={})
for num in numbers["numbers"]:
    print(f"  {num['number']} — expires {num['expires_at']}")
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

    // Initiate an AI voice call
    call, _ := mc.Post(ctx, "phone", "/voice/call", map[string]interface{}{
        "to":           "+14155551234",
        "task":         "You are an appointment reminder. Confirm dental appointment Tuesday at 2 PM.",
        "voice":        "nova",
        "max_duration": 120,
    })
    fmt.Printf("Call initiated: %s\n", call["call_id"])

    // Get transcript
    transcript, _ := mc.Call(ctx, "phone", fmt.Sprintf("/voice/call/%s", call["call_id"]))
    fmt.Printf("Status: %s, Duration: %v\n", transcript["status"], transcript["duration_seconds"])

    // Carrier lookup
    info, _ := mc.Post(ctx, "phone", "/lookup", map[string]interface{}{
        "phoneNumber": "+14155551234",
    })
    fmt.Printf("Carrier: %s, Type: %s\n", info["carrier"], info["line_type"])

    // Fraud risk assessment
    risk, _ := mc.Post(ctx, "phone", "/lookup/fraud", map[string]interface{}{
        "phoneNumber": "+14155551234",
    })
    fmt.Printf("Risk: %s (score: %v)\n", risk["risk_level"], risk["risk_score"])

    // Buy a number
    number, _ := mc.Post(ctx, "phone", "/numbers/buy", map[string]interface{}{
        "country":   "US",
        "area_code": "415",
    })
    fmt.Printf("Leased: %s\n", number["number"])

    // List owned numbers
    numbers, _ := mc.Post(ctx, "phone", "/numbers/list", map[string]interface{}{})
    fmt.Printf("Numbers: %v\n", numbers["numbers"])
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

    // Initiate an AI voice call
    call, _ := mc.Post(ctx, "phone", "/voice/call", map[string]interface{}{
        "to":           "+14155551234",
        "task":         "You are an appointment reminder. Confirm dental appointment Tuesday at 2 PM.",
        "voice":        "nova",
        "max_duration": 120,
    })
    fmt.Printf("Call initiated: %s\n", call["call_id"])

    // Get transcript
    transcript, _ := mc.Call(ctx, "phone", fmt.Sprintf("/voice/call/%s", call["call_id"]))
    fmt.Printf("Status: %s, Duration: %v\n", transcript["status"], transcript["duration_seconds"])

    // Carrier lookup
    info, _ := mc.Post(ctx, "phone", "/lookup", map[string]interface{}{
        "phoneNumber": "+14155551234",
    })
    fmt.Printf("Carrier: %s, Type: %s\n", info["carrier"], info["line_type"])

    // Fraud risk assessment
    risk, _ := mc.Post(ctx, "phone", "/lookup/fraud", map[string]interface{}{
        "phoneNumber": "+14155551234",
    })
    fmt.Printf("Risk: %s (score: %v)\n", risk["risk_level"], risk["risk_score"])

    // Buy a number
    number, _ := mc.Post(ctx, "phone", "/numbers/buy", map[string]interface{}{
        "country":   "US",
        "area_code": "415",
    })
    fmt.Printf("Leased: %s\n", number["number"])
}
```

:::

## Limitations

- **Outbound only** — inbound call handling and receiving calls is not supported
- **No SMS** — phone numbers are voice-only, SMS is not available
- **30-minute max call duration** — calls are automatically terminated at 1800 seconds
- **US/CA default** — numbers outside US and Canada require KYC verification
- **English only** — voice AI currently supports English language conversations only
- **30-day lease** — phone numbers auto-renew monthly unless cancelled
- **E.164 format required** — all phone numbers must include country code (e.g., `+14155551234`)
- **Caller ID** — the `from` field must be a number you've leased, or it will be omitted (calls show as unknown)
