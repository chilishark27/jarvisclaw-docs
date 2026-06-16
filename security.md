# Security

JarvisClaw is designed with defense-in-depth for API gateway security.

## API Key Security

- **Hashed storage** — API keys are stored as irreversible hashes. Raw keys are shown once at creation and never stored or logged.
- **Prefix identification** — Keys use a `sk-` prefix for identification without exposing the secret portion.
- **Key rotation** — Create new keys and revoke old ones instantly from the dashboard.

## On-Chain Verification (x402)

For agent payments, security is cryptographic:

- **Signature verification** — Every x402 payment is verified against the wallet's public key
- **Facilitator validation** — Payments are verified by the CDP facilitator before requests are processed
- **Replay protection** — Each payment includes a nonce to prevent double-spend
- **No custody** — JarvisClaw never holds your private keys

## Rate Limiting

Multiple layers of rate limiting protect against abuse:

| Layer | Scope |
|-------|-------|
| Per-key RPM | Requests per minute per API key |
| Per-IP | Limits requests from a single IP address |
| Daily spending | Caps total spend per account per day |
| Concurrent | Limits simultaneous in-flight requests |

Rate-limited requests return HTTP 429 with a `Retry-After` header.

## Encryption

- **TLS 1.2+** — All API traffic is encrypted in transit
- **Request/response bodies** — Never logged in plaintext
- **Wallet keys** — Only used client-side for signing; never transmitted to servers

## Best Practices

1. Never commit API keys to source control
2. Use environment variables for key storage
3. Set daily spending limits appropriate to your workload
4. Rotate keys periodically
5. Use separate keys for development and production
