# Compute API (Modal Sandbox)

Isolated code execution in secure cloud containers via Modal. Create sandboxes, execute arbitrary commands, manage lifecycle. Supports resource limits, network isolation, and automatic cleanup. Designed as a secure runtime for AI agent code execution.

**Base URL:** `https://api.jarvisclaw.ai/v1/marketplace/compute`

## Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/sandbox/create` | Create a new isolated sandbox |
| POST | `/sandbox/exec` | Execute a command in a sandbox |
| POST | `/sandbox/status` | Get sandbox status and metadata |
| POST | `/sandbox/terminate` | Terminate and clean up a sandbox |

## Pricing

| Endpoint | Price | Description |
|----------|-------|-------------|
| /sandbox/create | $0.01/sandbox | Provision a new container |
| /sandbox/exec | $0.001/request | Execute a command |
| /sandbox/status | $0.001/request | Query sandbox state |
| /sandbox/terminate | $0.001/request | Teardown and cleanup |

## POST /sandbox/create

Create a new isolated sandbox container.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `timeout` | integer | No | Max sandbox lifetime in seconds (max 300). Default: `300` |

### Request

```json
{
  "timeout": 300
}
```

### Response

```json
{
  "sandbox_id": "sb_a1b2c3d4e5",
  "status": "running",
  "created_at": "2026-06-13T10:00:00Z",
  "expires_at": "2026-06-13T10:05:00Z"
}
```

## POST /sandbox/exec

Run a command inside an existing sandbox.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sandbox_id` | string | Yes | Sandbox identifier |
| `command` | string | Yes | Shell command to execute |

### Request

```json
{
  "sandbox_id": "sb_a1b2c3d4e5",
  "command": "python -c \"import sys; print(sys.version)\""
}
```

### Response

```json
{
  "exit_code": 0,
  "stdout": "3.11.9 (main, Apr  2 2024, 08:25:04) [GCC 12.2.0]\n",
  "stderr": "",
  "duration_ms": 45
}
```

## POST /sandbox/status

Get current status and metadata of a sandbox.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sandbox_id` | string | Yes | Sandbox identifier |

### Request

```json
{
  "sandbox_id": "sb_a1b2c3d4e5"
}
```

### Response

```json
{
  "sandbox_id": "sb_a1b2c3d4e5",
  "status": "running",
  "created_at": "2026-06-13T10:00:00Z",
  "expires_at": "2026-06-13T10:05:00Z",
  "commands_executed": 3
}
```

## POST /sandbox/terminate

Immediately terminate a sandbox and release all resources.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sandbox_id` | string | Yes | Sandbox identifier |

### Request

```json
{
  "sandbox_id": "sb_a1b2c3d4e5"
}
```

### Response

```json
{
  "sandbox_id": "sb_a1b2c3d4e5",
  "status": "terminated"
}
```

## Errors

| HTTP Status | Code | Description |
|-------------|------|-------------|
| 404 | `sandbox_not_found` | The specified sandbox_id does not exist |
| 410 | `sandbox_terminated` | The sandbox has already been terminated |
| 408 | `sandbox_timeout` | The sandbox exceeded its timeout and was auto-terminated |
| 500 | `exec_failed` | Command execution failed due to an internal error |

## Code Examples

::: code-group

```bash [cURL]
# Create a sandbox
curl -X POST https://api.jarvisclaw.ai/v1/marketplace/compute/sandbox/create \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"timeout": 300}'

# Execute a command
curl -X POST https://api.jarvisclaw.ai/v1/marketplace/compute/sandbox/exec \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"sandbox_id": "sb_a1b2c3d4e5", "command": "python -c \"print(2+2)\""}'

# Check status
curl -X POST https://api.jarvisclaw.ai/v1/marketplace/compute/sandbox/status \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"sandbox_id": "sb_a1b2c3d4e5"}'

# Terminate
curl -X POST https://api.jarvisclaw.ai/v1/marketplace/compute/sandbox/terminate \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"sandbox_id": "sb_a1b2c3d4e5"}'
```

```python [Python (API Key)]
import requests

BASE = "https://api.jarvisclaw.ai/v1/marketplace/compute/sandbox"
HEADERS = {
    "Authorization": "Bearer sk-your-api-key",
    "Content-Type": "application/json",
}

# Create a sandbox
resp = requests.post(f"{BASE}/create", headers=HEADERS, json={
    "timeout": 300,
})
sandbox_id = resp.json()["sandbox_id"]
print(f"Created sandbox: {sandbox_id}")

# Execute a command
resp = requests.post(f"{BASE}/exec", headers=HEADERS, json={
    "sandbox_id": sandbox_id,
    "command": "python -c \"import math; print(math.pi)\"",
})
print(resp.json()["stdout"])

# Check status
resp = requests.post(f"{BASE}/status", headers=HEADERS, json={
    "sandbox_id": sandbox_id,
})
print(resp.json()["status"])

# Terminate when done
requests.post(f"{BASE}/terminate", headers=HEADERS, json={
    "sandbox_id": sandbox_id,
})
```

```python [Python (x402 Agent)]
from jarvisclaw import ComputeClient

# --- Option A: Base chain (EVM) ---
# Hex private key -> USDC on Base (Chain ID 8453)
compute = ComputeClient(private_key="0x<evm-private-key>")

# --- Option B: Solana ---
# Base58 keypair -> USDC SPL on Solana mainnet
# compute = ComputeClient(private_key="<solana-bs58-keypair>")

# SDK auto-detects chain from key format - no config needed

# Create a sandbox
sandbox = compute.create(timeout=300)
print(f"Sandbox ID: {sandbox.sandbox_id}")

# Execute commands
result = compute.exec(sandbox.sandbox_id, "pip install numpy && python -c \"import numpy; print(numpy.__version__)\"")
print(result.stdout)

result = compute.exec(sandbox.sandbox_id, "python -c \"import numpy as np; print(np.random.rand(3))\"")
print(result.stdout)

# Check status
status = compute.status(sandbox.sandbox_id)
print(f"Commands run: {status.commands_executed}")

# Terminate
compute.terminate(sandbox.sandbox_id)
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
    cc, _ := jc.NewComputeClient(jc.WithAPIKey("sk-your-api-key"))

    // Create a sandbox
    sandbox, _ := cc.Create(ctx, jc.WithTimeout(300))
    fmt.Printf("Sandbox ID: %s\n", sandbox.SandboxID)

    // Execute a command
    result, _ := cc.Exec(ctx, sandbox.SandboxID, "python -c \"print('hello')\"")
    fmt.Printf("Output: %s\n", result.Stdout)

    // Check status
    status, _ := cc.Status(ctx, sandbox.SandboxID)
    fmt.Printf("Status: %s, Commands: %d\n", status.Status, status.CommandsExecuted)

    // Terminate
    cc.Terminate(ctx, sandbox.SandboxID)
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

    // x402 Agent wallet - pays per-call via USDC on Base (Chain ID 8453)
    cc, _ := jc.NewComputeClient(jc.WithPrivateKey("0x<evm-private-key>"))

    // Create a sandbox
    sandbox, _ := cc.Create(ctx, jc.WithTimeout(300))
    fmt.Printf("Sandbox ID: %s\n", sandbox.SandboxID)

    // Execute commands
    result, _ := cc.Exec(ctx, sandbox.SandboxID, "python -c \"print('agent execution')\"")
    fmt.Printf("Output: %s\n", result.Stdout)

    // Terminate when done
    cc.Terminate(ctx, sandbox.SandboxID)
}
```

:::

## Limitations

- **300s max timeout** — sandboxes auto-terminate after the configured timeout (max 5 minutes)
- **No persistent storage** — all data is lost when the sandbox terminates
- **CPU only** — no GPU resources available
- **No inbound connections** — sandboxes cannot receive network traffic from outside
- **10MB max payload** — request and response bodies are capped at 10MB
- **No network egress** — sandboxes operate in an isolated network environment
