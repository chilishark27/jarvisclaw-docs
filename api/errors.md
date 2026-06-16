# Error Reference

Complete error code reference for all JarvisClaw API services. Includes HTTP status codes, x402 payment errors, and service-specific error codes.

**Base URL:** `https://api.jarvisclaw.ai/v1`

## HTTP Status Codes

| Code | Name | Description | Resolution |
|------|------|-------------|------------|
| 400 | Bad Request | Malformed request body or missing required parameters. | Check request format and required fields. |
| 401 | Unauthorized | Invalid or missing API key. | Verify your API key is correct and active. |
| 402 | Payment Required | x402 payment needed or insufficient USDC balance. | Top up USDC wallet or check payment signature. |
| 403 | Forbidden | API key lacks permission for this resource. | Check key permissions in dashboard. |
| 404 | Not Found | Endpoint or resource does not exist. | Verify the URL path is correct. |
| 429 | Rate Limited | Too many requests in time window. | Back off and retry after the Retry-After header value. |
| 500 | Internal Server Error | Unexpected server failure. | Retry with exponential backoff; contact support if persistent. |
| 502 | Bad Gateway | Upstream provider unavailable. | Retry; the provider may be temporarily down. |
| 503 | Service Unavailable | Service overloaded or in maintenance. | Wait and retry after a few seconds. |

## x402 Payment Errors

| Code | Name | Description | Resolution |
|------|------|-------------|------------|
| insufficient_balance | Insufficient Balance | USDC balance too low to cover request cost. | Top up your wallet with USDC (Base or Solana). |
| invalid_signature | Invalid Signature | Payment signature verification failed. | Check private key matches the wallet with USDC. |
| nonce_conflict | Nonce Conflict | Payment nonce already used (replay protection). | SDK handles automatically; if manual, increment nonce. |
| payment_expired | Payment Expired | Payment authorization timestamp too old. | Retry the request (SDK re-signs automatically). |
| chain_timeout | Chain Timeout | On-chain verification timed out. | Retry; blockchain may be congested. |
| amount_mismatch | Amount Mismatch | Signed amount doesn't match required payment. | Don't modify payment amount; let SDK calculate. |
| unsupported_network | Unsupported Network | Payment chain not supported. | Use Base (chain 8453) or Solana mainnet. |

## General API Errors

| Code | Name | Description | Resolution |
|------|------|-------------|------------|
| invalid_model | Invalid Model | Requested model not found or not available. | Check /v1/models for available models. |
| context_too_long | Context Too Long | Input exceeds model's context window. | Reduce message history or use a model with larger context. |
| content_filtered | Content Filtered | Request or response blocked by safety filters. | Modify the prompt to comply with usage policies. |
| upstream_error | Upstream Error | The upstream AI provider returned an error. | Retry; if persistent, try a different model. |

## Code Examples

::: code-group

```bash [cURL]
# Check HTTP status and error body
curl -s -w "\nHTTP_STATUS:%{http_code}" \
  https://api.jarvisclaw.ai/v1/chat/completions \
  -H "Authorization: Bearer $JARVISCLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-4o-mini","messages":[{"role":"user","content":"Hello"}]}' \
| tee >(tail -1 | grep -o "HTTP_STATUS:[0-9]*") \
| head -n -1 | jq .

# On error the body will contain:
# { "error": { "code": "invalid_model", "message": "..." } }
# or for x402:
# { "error": { "code": "insufficient_balance", "message": "..." } }
```

```python [Python]
import jarvisclaw
from jarvisclaw.errors import (
    AuthenticationError,
    RateLimitError,
    InsufficientBalanceError,
    APIError,
    JarvisClawError,
)
import time

client = jarvisclaw.Client(api_key="your-api-key")

def chat_with_retry(messages, model="gpt-4o-mini", max_retries=3):
    for attempt in range(max_retries):
        try:
            return client.chat.completions.create(
                model=model,
                messages=messages,
            )

        except AuthenticationError as e:
            # 401 — bad API key, no point retrying
            print(f"Authentication failed: {e.message}")
            raise

        except InsufficientBalanceError as e:
            # 402 — out of USDC; top up and retry manually
            print(f"Insufficient balance: {e.message}")
            print(f"Error code: {e.body.get('error', {}).get('code')}")
            raise

        except RateLimitError as e:
            # 429 — back off using Retry-After when available
            wait = e.retry_after or (2 ** attempt)
            print(f"Rate limited. Retrying in {wait}s …")
            time.sleep(wait)

        except APIError as e:
            # 500 / 502 / 503 — transient server errors
            if e.status_code in (500, 502, 503) and attempt < max_retries - 1:
                wait = 2 ** attempt
                print(f"Server error {e.status_code}. Retrying in {wait}s …")
                time.sleep(wait)
            else:
                # 400 / 403 / 404 or exhausted retries
                code = e.body.get("error", {}).get("code", "unknown")
                print(f"API error [{e.status_code}] {code}: {e.message}")
                raise

        except JarvisClawError as e:
            # x402 payment / SDK-level errors
            print(f"SDK error: {e}")
            raise

    raise RuntimeError("Max retries exceeded")
```

```go [Go]
package main

import (
	"context"
	"errors"
	"fmt"
	"time"

	jarvisclaw "github.com/api-jarvisclaw/go-sdk"
)

func chatWithRetry(
	ctx context.Context,
	client *jarvisclaw.Client,
	messages []jarvisclaw.Message,
	maxRetries int,
) (*jarvisclaw.ChatCompletion, error) {
	for attempt := range maxRetries {
		resp, err := client.Chat.Completions.Create(ctx, jarvisclaw.ChatCompletionRequest{
			Model:    "gpt-4o-mini",
			Messages: messages,
		})
		if err == nil {
			return resp, nil
		}

		var authErr *jarvisclaw.AuthenticationError
		if errors.As(err, &authErr) {
			// 401 — invalid API key, no retry
			return nil, fmt.Errorf("authentication failed: %w", err)
		}

		var balanceErr *jarvisclaw.InsufficientBalanceError
		if errors.As(err, &balanceErr) {
			// 402 — top up USDC wallet
			return nil, fmt.Errorf("insufficient balance: %w", err)
		}

		var rateErr *jarvisclaw.RateLimitError
		if errors.As(err, &rateErr) {
			// 429 — back off
			wait := time.Duration(1<<attempt) * time.Second
			fmt.Printf("Rate limited. Retrying in %s ...\n", wait)
			time.Sleep(wait)
			continue
		}

		var apiErr *jarvisclaw.APIError
		if errors.As(err, &apiErr) {
			switch apiErr.StatusCode {
			case 500, 502, 503:
				// Transient server error — exponential backoff
				if attempt < maxRetries-1 {
					wait := time.Duration(1<<attempt) * time.Second
					fmt.Printf("Server error %d. Retrying in %s ...\n", apiErr.StatusCode, wait)
					time.Sleep(wait)
					continue
				}
			default:
				// 400 / 403 / 404 / upstream_error — not retryable
				code := apiErr.Body["error"]
				return nil, fmt.Errorf("api error [%d] %v: %s", apiErr.StatusCode, code, apiErr.Message)
			}
		}

		// PaymentError / other SDK errors
		return nil, fmt.Errorf("sdk error: %w", err)
	}
	return nil, fmt.Errorf("max retries (%d) exceeded", maxRetries)
}
```

:::
---
