# SDK Reference

Official SDKs for JarvisClaw AI API Gateway.

## Python SDK

```bash
pip install jarvisclaw
```

### Chat Completion

```python
from jarvisclaw import Client

client = Client(api_key="sk-your-api-key")

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}],
)
print(response.choices[0].message.content)
```

### Image Generation

```python
response = client.images.generate(
    model="dall-e-3",
    prompt="A cyberpunk cityscape at sunset",
    size="1024x1024",
)
print(response.data[0].url)
```

### Video Generation

```python
# Video generation is async — submit then poll
job = client.videos.generate(
    model="kling-v2",
    prompt="A cat walking on the moon",
)

# Poll until complete
result = client.videos.get(job.id)
print(result.url)
```

### Agent Payment (x402)

```python
from jarvisclaw import AgentClient

# No API key needed — pay per call with USDC
client = AgentClient(private_key="0x_your_base_private_key")

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}],
)
```

## Go SDK

```bash
go get github.com/api-jarvisclaw/go-sdk
```

### Chat Completion

```go
package main

import (
    "context"
    "fmt"
    jc "github.com/api-jarvisclaw/go-sdk"
)

func main() {
    client := jc.New(jc.WithAPIKey("sk-your-api-key"))

    resp, err := client.Chat(context.Background(), &jc.ChatRequest{
        Model:    "gpt-4o",
        Messages: []jc.Message{{Role: "user", Content: "Hello!"}},
    })
    if err != nil {
        panic(err)
    }
    fmt.Println(resp.Choices[0].Message.Content)
}
```

### Image Generation

```go
resp, err := client.ImageGenerate(ctx, &jc.ImageRequest{
    Model:  "dall-e-3",
    Prompt: "A cyberpunk cityscape at sunset",
    Size:   "1024x1024",
})
fmt.Println(resp.Data[0].URL)
```

### Video Generation

```go
job, err := client.VideoGenerate(ctx, &jc.VideoRequest{
    Model:  "kling-v2",
    Prompt: "A cat walking on the moon",
})

// Poll for result
result, err := client.VideoGet(ctx, job.ID)
fmt.Println(result.URL)
```

### Agent Payment (x402)

```go
client := jc.New(jc.WithPrivateKey("0x_your_base_private_key"))

resp, err := client.Chat(ctx, &jc.ChatRequest{
    Model:    "gpt-4o",
    Messages: []jc.Message{{Role: "user", Content: "Hello!"}},
})
```

## OpenAI SDK Compatibility

JarvisClaw is OpenAI-compatible. You can use the official OpenAI SDK directly:

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.jarvisclaw.ai/v1",
    api_key="sk-your-api-key",
)
```
