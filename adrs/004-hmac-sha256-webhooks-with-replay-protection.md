# ADR 004 — HMAC-SHA256 webhooks with replay protection

- **Status:** accepted
- **Date:** 2026-04-27

## Context

Webhooks let LIQAA notify merchant servers when something happens (call started, ended, recording ready). The merchant server needs to know **the event came from us, not an attacker spamming the endpoint**.

Three options:
1. mTLS (most secure, most painful to set up)
2. Bearer token in `Authorization: Bearer …`
3. HMAC signature in a header — Stripe-style

## Decision

Every webhook request includes:

```
LIQAA-Signature: t=1714680000,v1=<hex>
```

where `<hex> = HMAC-SHA256(secret, "<timestamp>.<raw_body>")`.

The merchant verifies by:
1. Splitting the header into `t` and `v1`.
2. Computing the same HMAC.
3. Comparing in **constant time** (`crypto.timingSafeEqual` or equivalent).
4. Rejecting if `|now - t| > 5 minutes` (replay protection).

We retry failed deliveries with exponential backoff: 1m, 5m, 30m, 2h, 12h (5 attempts total).

## Consequences

### Positive
- **No bearer tokens to steal.** The signature is per-request.
- **Replay-resistant.** Window is 5 min by default; merchants can tighten.
- **Stripe-compatible mental model.** Most devs already know `t=…,v1=…`.
- **No mTLS complexity.** Works through every proxy and load balancer.

### Negative
- **Constant-time compare is easy to get wrong.** We document it heavily and ship `WebhookVerifier` helpers in every SDK ([`liqaa-php`](https://github.com/hartemyaakoub/liqaa-php), [`liqaa-python`](https://github.com/hartemyaakoub/liqaa-python), [`liqaa-go`](https://github.com/hartemyaakoub/liqaa-go), [`@liqaa/js`](https://github.com/hartemyaakoub/liqaa-js)).

### Mitigations
- Each SDK ships a `verify(headers, rawBody, secret)` function. Merchants are encouraged to use it.
- The Console UI ships a "test webhook" button that delivers a real payload with a real signature.

## References
- [Stripe webhook signing](https://docs.stripe.com/webhooks#verify-events)
- [GitHub webhook signing](https://docs.github.com/en/webhooks/using-webhooks/validating-webhook-deliveries)
