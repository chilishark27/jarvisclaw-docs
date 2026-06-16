# Smart Router

JarvisClaw's smart router lets you use routing aliases instead of specifying a model directly. The router selects the optimal model based on your intent.

## Routing Aliases

| Alias | Behavior |
|-------|----------|
| `auto` | Picks the best available model balancing quality and cost |
| `free` | Routes to free-tier models only (no charge) |
| `eco` | Selects the most cost-efficient paid model |
| `premium` | Routes to the highest-capability model available |

Aliases work anywhere you'd normally pass a model name — chat, images, embeddings.

## How It Works

When you send `model: "auto"`, the router evaluates:
- Task complexity (from prompt length and structure)
- Available models and their current latency
- Cost-performance tradeoff for the selected tier

The actual model used is returned in the response's `model` field.

## Example

::: code-group

```python [Python]
from openai import OpenAI

client = OpenAI(
    base_url="https://api.jarvisclaw.ai/v1",
    api_key="sk-your-api-key",
)

# Let the router pick the best model
response = client.chat.completions.create(
    model="auto",
    messages=[{"role": "user", "content": "Summarize this article..."}],
)

# See which model was actually used
print(f"Routed to: {response.model}")
print(response.choices[0].message.content)
```

```bash [cURL]
curl https://api.jarvisclaw.ai/v1/chat/completions \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "eco",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

:::
