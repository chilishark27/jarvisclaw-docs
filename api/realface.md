# RealFace & Virtual Portrait API

Enroll real people (with liveness verification) or synthetic characters for consistent identity in AI-generated videos. Creates a reusable `ta_xxx` asset_id for Seedance 2.0 video generation.

## Base URL

```
https://api.jarvisclaw.ai/v1/marketplace/realface
```

## Endpoints

| Method | Endpoint | Description | Price |
|--------|----------|-------------|-------|
| POST | `/realface/init` | Initialize liveness session | Free (10/hour limit) |
| GET | `/realface/status` | Poll session status | Free |
| POST | `/realface/enroll` | Enroll verified face | $0.01 |
| POST | `/portrait/enroll` | Enroll virtual character | $0.01 |

## How It Works

1. **Init** — Create a liveness session, receive a QR code link
2. **User scans QR** — Completes nod + blink challenge on phone camera (120s expiry)
3. **Poll status** — Wait for liveness verification to complete
4. **Enroll** — Submit face image + completed session to get a reusable `asset_id`
5. **Generate video** — Pass `asset_id` to Seedance 2.0 / 2.0-fast for identity-consistent output

For virtual characters (illustrations, avatars, 3D renders), skip steps 1-3 and use `/portrait/enroll` directly.

---

## Initialize Liveness Session

`POST /v1/marketplace/realface/init`

Start a phone-based liveness verification session. Returns a QR code URL that the subject scans to complete the challenge.

### Request

```json
{
  "name": "John Doe"
}
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Display name for the enrolled identity |
| `redirect_url` | string | No | URL to redirect user after liveness completion |
| `webhook_url` | string | No | URL for server-to-server completion callback |

### Response

```json
{
  "group_id": "grp_a1b2c3d4",
  "name": "John Doe",
  "verification_url": "https://verify.jarvisclaw.ai/liveness/grp_a1b2c3d4",
  "qr_code_url": "https://verify.jarvisclaw.ai/qr/grp_a1b2c3d4.png",
  "expires_at": "2025-06-01T12:02:00Z",
  "status": "pending"
}
```

::: warning QR Link Expiry
The verification URL expires after **120 seconds**. If the user doesn't complete the challenge in time, call `/init` again.
:::

---

## Poll Session Status

`GET /v1/marketplace/realface/status`

Check whether the user has completed the liveness challenge.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `groupId` | string | Yes | Group ID from init response |

### Response

```json
{
  "group_id": "grp_a1b2c3d4",
  "name": "John Doe",
  "status": "completed",
  "liveness_score": 0.98
}
```

Status values: `pending`, `in_progress`, `completed`, `expired`, `failed`

---

## Enroll Verified Face

`POST /v1/marketplace/realface/enroll`

After liveness is completed, submit a reference photo to finalize enrollment. The photo must match the person who completed the liveness challenge.

### Request

```json
{
  "name": "John Doe",
  "image_url": "https://example.com/john-reference.jpg",
  "group_id": "grp_a1b2c3d4"
}
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Display name (must match init name) |
| `image_url` | string | Yes | Public HTTPS URL of reference face photo |
| `group_id` | string | Yes | Group ID from completed liveness session |

### Response

```json
{
  "asset_id": "ta_rf_xyz789",
  "name": "John Doe",
  "group_id": "grp_a1b2c3d4",
  "liveness_verified": true,
  "created_at": "2025-06-01T12:05:00Z"
}
```

---

## Enroll Virtual Portrait

`POST /v1/marketplace/portrait/enroll`

Enroll a virtual character (illustration, avatar, 3D render, AI-generated face) without liveness verification. No identity check — use for fictional characters.

### Request

```json
{
  "name": "Cyber Knight",
  "image_url": "https://example.com/character-portrait.png"
}
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Character name |
| `image_url` | string | Yes | Public HTTPS URL of character image |

### Response

```json
{
  "asset_id": "ta_pt_def456",
  "name": "Cyber Knight",
  "liveness_verified": false,
  "created_at": "2025-06-01T12:05:00Z"
}
```

---

## Examples

::: code-group

```bash [cURL]
# 1. Initialize liveness session
curl -X POST https://api.jarvisclaw.ai/v1/marketplace/realface/init \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"name": "John Doe"}'

# 2. Poll status
curl "https://api.jarvisclaw.ai/v1/marketplace/realface/status?groupId=grp_a1b2c3d4" \
  -H "Authorization: Bearer sk-your-api-key"

# 3. Enroll verified face
curl -X POST https://api.jarvisclaw.ai/v1/marketplace/realface/enroll \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "image_url": "https://example.com/john-photo.jpg",
    "group_id": "grp_a1b2c3d4"
  }'

# Virtual portrait (no liveness needed)
curl -X POST https://api.jarvisclaw.ai/v1/marketplace/portrait/enroll \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Cyber Knight",
    "image_url": "https://example.com/character.png"
  }'
```

```python [Python (API Key)]
import requests
import time

BASE_RF = "https://api.jarvisclaw.ai/v1/marketplace/realface"
BASE_PT = "https://api.jarvisclaw.ai/v1/marketplace/portrait"
HEADERS = {
    "Authorization": "Bearer sk-your-api-key",
    "Content-Type": "application/json",
}

# --- RealFace Flow ---

# 1. Initialize liveness session
resp = requests.post(f"{BASE_RF}/init", headers=HEADERS, json={
    "name": "John Doe",
})
session = resp.json()
print(f"QR Code: {session['qr_code_url']}")
print(f"Expires: {session['expires_at']}")

# 2. Poll until completed (120s max)
group_id = session["group_id"]
while True:
    resp = requests.get(f"{BASE_RF}/status", headers=HEADERS, params={
        "groupId": group_id,
    })
    status = resp.json()
    if status["status"] == "completed":
        print(f"Liveness verified (score: {status['liveness_score']})")
        break
    elif status["status"] in ("expired", "failed"):
        raise Exception(f"Liveness failed: {status['status']}")
    time.sleep(3)

# 3. Enroll face
resp = requests.post(f"{BASE_RF}/enroll", headers=HEADERS, json={
    "name": "John Doe",
    "image_url": "https://example.com/john-photo.jpg",
    "group_id": group_id,
})
asset = resp.json()
print(f"Asset ID for Seedance 2.0: {asset['asset_id']}")

# --- Virtual Portrait (skip liveness) ---

resp = requests.post(f"{BASE_PT}/enroll", headers=HEADERS, json={
    "name": "Cyber Knight",
    "image_url": "https://example.com/character.png",
})
print(f"Portrait asset: {resp.json()['asset_id']}")
```

```python [Python (x402 Agent)]
from jarvisclaw import Client

# x402 agent — pays per call with USDC, no API key needed
client = Client(private_key="0x<agent-wallet-private-key>")

# 1. Initialize liveness session
session = client.post("/v1/marketplace/realface/init", json={
    "name": "John Doe",
})
print(f"QR Code: {session['qr_code_url']}")

# 2. Poll status
import time
while True:
    status = client.get("/v1/marketplace/realface/status", params={
        "groupId": session["group_id"],
    })
    if status["status"] == "completed":
        break
    time.sleep(3)

# 3. Enroll
asset = client.post("/v1/marketplace/realface/enroll", json={
    "name": "John Doe",
    "image_url": "https://example.com/john-photo.jpg",
    "group_id": session["group_id"],
})
print(f"Asset ID: {asset['asset_id']}")

# Virtual portrait
portrait = client.post("/v1/marketplace/portrait/enroll", json={
    "name": "Cyber Knight",
    "image_url": "https://example.com/character.png",
})
print(f"Portrait: {portrait['asset_id']}")
```

```go [Go (API Key)]
package main

import (
    "fmt"
    "time"

    jarvisclaw "github.com/api-jarvisclaw/go-sdk"
)

func main() {
    client := jarvisclaw.NewClient(jarvisclaw.WithAPIKey("sk-your-api-key"))

    // 1. Initialize liveness session
    var session struct {
        GroupID         string `json:"group_id"`
        QRCodeURL       string `json:"qr_code_url"`
        VerificationURL string `json:"verification_url"`
        ExpiresAt       string `json:"expires_at"`
    }
    err := client.PostJSON("/v1/marketplace/realface/init", map[string]string{
        "name": "John Doe",
    }, &session)
    if err != nil {
        panic(err)
    }
    fmt.Printf("QR Code: %s\n", session.QRCodeURL)

    // 2. Poll until completed
    for {
        var status struct {
            Status        string  `json:"status"`
            LivenessScore float64 `json:"liveness_score"`
        }
        err = client.GetJSON("/v1/marketplace/realface/status", map[string]string{
            "groupId": session.GroupID,
        }, &status)
        if err != nil {
            panic(err)
        }
        if status.Status == "completed" {
            fmt.Printf("Liveness score: %.2f\n", status.LivenessScore)
            break
        }
        time.Sleep(3 * time.Second)
    }

    // 3. Enroll face
    var asset struct {
        AssetID string `json:"asset_id"`
    }
    err = client.PostJSON("/v1/marketplace/realface/enroll", map[string]string{
        "name":      "John Doe",
        "image_url": "https://example.com/john-photo.jpg",
        "group_id":  session.GroupID,
    }, &asset)
    if err != nil {
        panic(err)
    }
    fmt.Printf("Asset ID: %s\n", asset.AssetID)
}
```

```go [Go (x402 Agent)]
package main

import (
    "fmt"
    "time"

    jarvisclaw "github.com/api-jarvisclaw/go-sdk"
)

func main() {
    // x402 agent — pays per call with USDC, no API key needed
    client, err := jarvisclaw.NewClient(
        jarvisclaw.WithPrivateKey("0x<agent-wallet-private-key>"),
    )
    if err != nil {
        panic(err)
    }

    // 1. Initialize liveness session
    var session struct {
        GroupID   string `json:"group_id"`
        QRCodeURL string `json:"qr_code_url"`
    }
    _ = client.PostJSON("/v1/marketplace/realface/init", map[string]string{
        "name": "John Doe",
    }, &session)
    fmt.Printf("QR: %s\n", session.QRCodeURL)

    // 2. Poll status
    for {
        var status struct {
            Status string `json:"status"`
        }
        _ = client.GetJSON("/v1/marketplace/realface/status", map[string]string{
            "groupId": session.GroupID,
        }, &status)
        if status.Status == "completed" {
            break
        }
        time.Sleep(3 * time.Second)
    }

    // 3. Enroll
    var asset struct {
        AssetID string `json:"asset_id"`
    }
    _ = client.PostJSON("/v1/marketplace/realface/enroll", map[string]string{
        "name":      "John Doe",
        "image_url": "https://example.com/john-photo.jpg",
        "group_id":  session.GroupID,
    }, &asset)
    fmt.Printf("Asset ID: %s\n", asset.AssetID)
}
```

:::

---

## Errors

| Code | Error | Description |
|------|-------|-------------|
| 425 | `liveness_incomplete` | Liveness challenge not yet completed — keep polling |
| 422 | `face_match_failed` | Enrolled photo doesn't match the person who completed liveness |
| 429 | `rate_limited` | Exceeded 10 init sessions per hour |
| 400 | `invalid_image_url` | Image URL is not a valid public HTTPS URL or returned non-200 |
| 400 | `session_expired` | Liveness session expired (120s timeout) |
| 404 | `group_not_found` | Invalid or unknown group_id |

---

## Limitations

- **Seedance only** — Asset IDs only work with Seedance 2.0 and Seedance 2.0-fast models
- **120s QR expiry** — Verification link expires quickly; user must scan and complete promptly
- **Public HTTPS only** — `image_url` must be a publicly accessible HTTPS URL (no localhost, no HTTP)
- **No celebrity enrollment** — Cannot enroll public figures without their active participation in the liveness challenge
- **No deletion** — Once enrolled, asset IDs are permanent and cannot be revoked
- **Virtual portraits skip verification** — `/portrait/enroll` does not verify identity; anyone can enroll any image
- **Single face per image** — Reference photo must contain exactly one clearly visible face
- **Consent required** — RealFace is designed for consent-based enrollment only; the enrolled person must complete the liveness challenge themselves
