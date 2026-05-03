# ADR 002 — Adopt Stripe-style `pk_live_` / `sk_live_` key separation

- **Status:** accepted
- **Date:** 2026-04-26
- **Decision-makers:** Yaakoub HARTEM
- **Supersedes:** —

## Context

When a developer integrates a video API into a webpage, code runs in two places:

1. **The browser** — visible to the world. Anything you put here can be opened in DevTools and copied.
2. **The server** — opaque. Customer secrets live here.

If we issue a single API key, developers will paste it into JavaScript and it will leak. This is not hypothetical: GitHub's [secret scanning logs](https://docs.github.com/en/code-security/secret-scanning) show thousands of secrets pushed to public repos every day.

Stripe solved this in 2011 with `pk_live_` (publishable, browser-safe) and `sk_live_` (secret, server-only). The pattern is now industry standard — Plaid, OpenAI, Mux, LiveKit Cloud all copy it.

## Decision

LIQAA issues **two keys per project**:

- `pk_live_xxxxx` — safe to ship in browser bundles. Can only call `/sdk-token` to exchange a per-user JWT.
- `sk_live_xxxxx` — server-only. Can call all admin endpoints (rooms, webhooks, billing).

JWTs returned by `/sdk-token` are scoped (room, user, expiry, permissions) and live 60 minutes by default.

## Consequences

### Positive
- **Familiar.** Every developer who has used Stripe understands this in 5 seconds.
- **Defense in depth.** Even if `pk_live_` leaks, the blast radius is "anyone can call /sdk-token for your project" — and we rate-limit that.
- **Easy to rotate.** Two keys, rotated independently.
- **Auto-detected by secret scanners.** GitHub, GitLab, and `trufflehog` all match the `sk_live_` regex pattern automatically.

### Negative
- **Two-step integration** instead of one. Devs need a server endpoint that calls `/sdk-token`. We provide framework starters (Next.js, Express, Laravel, Django) so this is a 30-second copy-paste.

### Mitigations
- See [liqaa-template-nextjs](https://github.com/hartemyaakoub/liqaa-template-nextjs) for the canonical token-exchange pattern.

## References
- [Stripe Keys docs](https://docs.stripe.com/keys)
- [TruffleHog detector list](https://github.com/trufflesecurity/trufflehog)
