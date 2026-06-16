# Quick Start

3 steps to start using JarvisClaw:

## 1. Get an API Key

Sign up at [api.jarvisclaw.ai](https://api.jarvisclaw.ai) and create an API key from the dashboard.

Or skip this — use [x402 direct payment](/x402) with just a USDC wallet.

## 2. Set Base URL

```
https://api.jarvisclaw.ai/v1
```

## 3. Make a Request

::: code-group

```python [Python]
from openai import OpenAI

client = OpenAI(
    base_url="https://api.jarvisclaw.ai/v1",
    api_key="sk-your-api-key",
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}],
)
print(response.choices[0].message.content)
```

```bash [cURL]
curl https://api.jarvisclaw.ai/v1/chat/completions \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

```go [Go]
package main

import (
    "context"
    "fmt"
    jc "github.com/api-jarvisclaw/go-sdk"
)

func main() {
    client := jc.New(jc.WithAPIKey("sk-your-api-key"))
    resp, _ := client.Chat(context.Background(), &jc.ChatRequest{
        Model:    "gpt-4o",
        Messages: []jc.Message{{Role: "user", Content: "Hello!"}},
    })
    fmt.Println(resp.Choices[0].Message.Content)
}
```

:::

## Smart Router

Set `model` to a routing alias instead of a specific model:

| Alias | Behavior |
|-------|----------|
| `auto` | Picks the best model for quality and cost |
| `free` | Free-tier models only |
| `eco` | Most cost-efficient paid model |
| `premium` | Highest capability model |

```python
response = client.chat.completions.create(
    model="auto",  # Smart Router picks the best model
    messages=[{"role": "user", "content": "Explain quantum computing"}],
)
```
