# Error Codes

Standard error responses and how to handle them.

## Error Response Format

All errors return a JSON body with this structure:

```json
{
  "error": {
    "message": "Description of what went wrong",
    "type": "invalid_request_error",
    "code": "invalid_api_key"
  }
}
```

## HTTP Status Codes

| Code | Meaning | Action |
|------|---------|--------|
| `400` | Bad Request — invalid parameters | Check request body and parameters |
| `401` | Unauthorized — invalid or missing API key | Verify your API key |
| `402` | Payment Required — x402 payment needed | Sign and attach payment (see x402 docs) |
| `403` | Forbidden — access denied to this resource | Check key permissions |
| `404` | Not Found — model or endpoint doesn't exist | Verify model ID and endpoint path |
| `429` | Rate Limited — too many requests | Back off and retry after `Retry-After` seconds |
| `500` | Internal Error — server-side failure | Retry with exponential backoff |
| `502` | Bad Gateway — upstream provider error | Retry or try a different model |
| `503` | Service Unavailable — temporarily overloaded | Retry after a brief delay |

## Error Types

| Type | Description |
|------|-------------|
| `invalid_request_error` | Malformed request, missing fields, invalid values |
| `authentication_error` | API key invalid, expired, or missing |
| `permission_error` | Key lacks access to the requested resource |
| `rate_limit_error` | Request rate or daily quota exceeded |
| `insufficient_balance` | Account balance too low for this request |
| `model_not_found` | Requested model doesn't exist or is unavailable |
| `upstream_error` | The upstream AI provider returned an error |
| `server_error` | Internal server error |

## Handling Errors

::: code-group

```python [Python]
from openai import OpenAI, APIError, RateLimitError

client = OpenAI(
    base_url="https://api.jarvisclaw.ai/v1",
    api_key="sk-your-api-key",
)

try:
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": "Hello"}],
    )
except RateLimitError:
    # Back off and retry
    print("Rate limited — waiting before retry")
except APIError as e:
    print(f"API error {e.status_code}: {e.message}")
```

```go [Go]
resp, err := client.Chat(ctx, &jc.ChatRequest{
    Model:    "gpt-4o",
    Messages: []jc.Message{{Role: "user", Content: "Hello"}},
})
if err != nil {
    var apiErr *jc.APIError
    if errors.As(err, &apiErr) {
        switch apiErr.StatusCode {
        case 429:
            // Rate limited — retry with backoff
        case 402:
            // Insufficient balance or payment needed
        default:
            log.Printf("API error %d: %s", apiErr.StatusCode, apiErr.Message)
        }
    }
}
```

:::

## Retry Strategy

For transient errors (429, 500, 502, 503), use exponential backoff:

1. Wait 1 second, retry
2. Wait 2 seconds, retry
3. Wait 4 seconds, retry
4. Give up after 3-5 attempts

The `Retry-After` header (when present) tells you exactly how long to wait.
