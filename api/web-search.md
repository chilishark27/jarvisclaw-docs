# Web Search

`POST /v1/search`

Search the web and get structured results.

## Request

```json
{
  "model": "search",
  "query": "latest advances in quantum computing 2025",
  "max_results": 5
}
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | Yes | Search model identifier |
| `query` | string | Yes | Search query string |
| `max_results` | integer | No | Maximum number of results to return. Default: `5` |

## Response

```json
{
  "results": [
    {
      "title": "Quantum Computing Breakthrough...",
      "url": "https://example.com/article",
      "snippet": "Researchers have achieved...",
      "published": "2025-03-15"
    }
  ],
  "query": "latest advances in quantum computing 2025"
}
```

## Examples

::: code-group

```python [Python]
from jarvisclaw import Client

client = Client(api_key="sk-your-api-key")

results = client.search(
    query="best practices for LLM prompt engineering",
    max_results=5,
)

for result in results:
    print(f"{result.title}: {result.url}")
```

```bash [cURL]
curl -X POST https://api.jarvisclaw.ai/v1/search \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "search",
    "query": "best practices for LLM prompt engineering",
    "max_results": 5
  }'
```

:::

## Use Cases

- Retrieval-augmented generation (RAG) — ground LLM responses in fresh web data
- Agent research — let agents gather real-time information
- Fact-checking — verify claims against current sources
