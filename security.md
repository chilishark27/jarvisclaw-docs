# Security

Security is built into every layer of the platform. Below is a summary of the key practices in place.

## API keys hashed at rest

All API keys are stored as SHA-256 hashes. The plaintext key is shown once at creation and never persisted.

## x402 payments verified on-chain

Every x402 payment is verified against the blockchain via a facilitator contract before the request is processed. No off-chain trust required.

## Rate limiting at every layer

Requests are rate-limited per user and per IP address. A global circuit breaker protects upstream providers from cascading failures.

## HD wallet keys shard-encrypted

HD wallet private keys are encrypted using shard-based key splitting. No single storage location holds a complete key.

## All traffic over TLS 1.3

Every connection is served over HTTPS with TLS 1.3, enforced at the Cloudflare edge. Plain HTTP is not accepted.
