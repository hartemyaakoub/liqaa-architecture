# ADR 003 — JWT token exchange instead of long-lived browser keys

- **Status:** accepted
- **Date:** 2026-04-26

## Context

The browser SDK needs to authenticate against our LiveKit signaling server. Two options:

1. Embed a long-lived API key in the SDK (`pk_live_…`) and let it talk directly to LiveKit.
2. Have the merchant's server issue a short-lived JWT scoped to one user and one room.

Option 1 is simpler but the key never expires. If a malicious user grabs it, they can join any room on the project forever.

## Decision

The browser SDK calls **`POST /sdk-token`** on the merchant's server (server-to-server, signed with `sk_live_`) to get a JWT scoped to:

- a single user identity (`sub: user@example.com`)
- one or more rooms (`rooms: ['support']`)
- expiry (`exp`: 60 minutes by default, max 24h)
- explicit permissions (`canPublish`, `canSubscribe`, `canPublishData`)

The SDK presents this JWT to LiveKit. LiveKit verifies the HMAC against our shared secret and admits the participant.

## Consequences

### Positive
- **Time-bounded blast radius.** A leaked JWT is dead in 60 minutes.
- **Scoped.** A JWT for `support` cannot join `private-meeting`.
- **No browser secrets.** `pk_live_` is the only thing in JS, and it can't do anything except call `/sdk-token` (rate-limited).
- **Auditable.** Every JWT issuance is logged on the merchant side.

### Negative
- **Extra hop.** First call adds ~50–200ms (token exchange). We mitigate by caching JWTs server-side until 5 min before expiry.
- **Merchants must run a server.** Cannot use LIQAA from a fully-static site (e.g., GitHub Pages alone). For static sites we offer a managed token endpoint at `/embed/{room}` (white-label hosted).

## References
- [RFC 7519: JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519)
- [LiveKit access tokens](https://docs.livekit.io/home/get-started/authentication/)
