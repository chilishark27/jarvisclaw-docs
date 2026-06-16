# Async & Concurrent Usage

Patterns for high-throughput and non-blocking API usage.

## Python — asyncio

The `jarvisclaw` SDK ships with an async client for use with `asyncio`:

```python
import asyncio
from jarvisclaw.aio import AsyncClient

client = AsyncClient(api_key="sk-your-api-key")

async def main():
    response = await client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": "Hello!"}],
    )
    print(response.choices[0].message.content)

asyncio.run(main())
```

### Concurrent Requests

```python
import asyncio
from jarvisclaw.aio import AsyncClient

client = AsyncClient(api_key="sk-your-api-key")

async def ask(question: str):
    resp = await client.chat.completions.create(
        model="auto",
        messages=[{"role": "user", "content": question}],
    )
    return resp.choices[0].message.content

async def main():
    questions = [
        "What is quantum computing?",
        "Explain neural networks",
        "What is blockchain?",
    ]
    # Run all requests concurrently
    results = await asyncio.gather(*[ask(q) for q in questions])
    for q, a in zip(questions, results):
        print(f"Q: {q}\nA: {a}\n")

asyncio.run(main())
```

### Async Streaming

```python
async def stream_response():
    stream = await client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": "Write a poem"}],
        stream=True,
    )
    async for chunk in stream:
        if chunk.choices[0].delta.content:
            print(chunk.choices[0].delta.content, end="")

asyncio.run(stream_response())
```

## Go — Goroutines

The Go SDK is safe for concurrent use across goroutines:

```go
package main

import (
    "context"
    "fmt"
    "sync"

    jc "github.com/api-jarvisclaw/go-sdk"
)

func main() {
    client := jc.New(jc.WithAPIKey("sk-your-api-key"))

    questions := []string{
        "What is quantum computing?",
        "Explain neural networks",
        "What is blockchain?",
    }

    var wg sync.WaitGroup
    for _, q := range questions {
        wg.Add(1)
        go func(question string) {
            defer wg.Done()
            resp, err := client.Chat(context.Background(), &jc.ChatRequest{
                Model:    "auto",
                Messages: []jc.Message{{Role: "user", Content: question}},
            })
            if err != nil {
                fmt.Printf("Error: %v\n", err)
                return
            }
            fmt.Printf("Q: %s\nA: %s\n\n", question, resp.Choices[0].Message.Content)
        }(q)
    }
    wg.Wait()
}
```

### Go Streaming

```go
stream, err := client.ChatStream(ctx, &jc.ChatRequest{
    Model:    "gpt-4o",
    Messages: []jc.Message{{Role: "user", Content: "Write a poem"}},
})
if err != nil {
    panic(err)
}
defer stream.Close()

for stream.Next() {
    chunk := stream.Current()
    fmt.Print(chunk.Choices[0].Delta.Content)
}
```

## Rate Limits

Concurrent requests share your account's rate limit. If you hit limits, the API returns HTTP 429. Implement exponential backoff or use the SDK's built-in retry logic.
