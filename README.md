<div align="center">

# LIQAA Architecture

**The decisions, the diagrams, the reasoning. Public.**

</div>

---

This repo holds the **public engineering record** for the LIQAA platform — the design decisions we've made, the trade-offs we considered, and the architecture diagrams that explain how the system actually works.

We publish this for two reasons:

1. **Trust.** Customers betting their product on LIQAA deserve to see how the sausage is made.
2. **Education.** "Build a Stripe-grade WebRTC SaaS from Algeria" is a non-trivial engineering challenge. Maybe these ADRs help someone else.

If you've ever read [Linear's spec docs](https://linear.app/method) or [Vercel's eng blog](https://vercel.com/blog), this is our version of that.

## Table of contents

### Architecture Decision Records (ADRs)

ADRs follow the [Michael Nygard format](https://github.com/joelparkerhenderson/architecture-decision-record). Status of each is one of: `proposed`, `accepted`, `superseded`, `deprecated`.

| #    | Title                                                                                                | Status     |
| ---- | ---------------------------------------------------------------------------------------------------- | ---------- |
| [001](./adrs/001-livekit-sfu-vs-build-our-own.md) | Use LiveKit SFU instead of building our own                       | accepted   |
| [002](./adrs/002-stripe-grade-pk-sk-separation.md) | pk_live / sk_live separation, Stripe-style                       | accepted   |
| [003](./adrs/003-persistent-rooms-by-conversation-id.md) | Persistent rooms via external_conversation_id                | accepted   |
| [004](./adrs/004-hmac-signed-webhooks-with-replay-protection.md) | HMAC-signed webhooks with 5-min replay window         | accepted   |
| [005](./adrs/005-jwt-sdk-tokens-instead-of-bearer-pk.md) | 1-hour JWT for browser sessions, no raw pk in DOM            | accepted   |
| [006](./adrs/006-mcp-server-for-ai-agents.md) | Ship an MCP server alongside the SDKs                                   | accepted   |
| [007](./adrs/007-publish-changelog-as-separate-repo.md) | Publish the changelog as its own GitHub repo                  | accepted   |
| [008](./adrs/008-public-benchmarks-as-a-feature.md) | Treat open benchmarks as a competitive feature                    | accepted   |
| [009](./adrs/009-rtl-and-arabic-as-tier-1.md) | Arabic + RTL as a tier-1 (not "i18n later") concern                     | accepted   |
| [010](./adrs/010-self-hostable-on-day-1.md) | Architect every service to be self-hostable from day one                  | accepted   |

### System diagrams

| Diagram | What it shows |
| --- | --- |
| [01 — Top-level data flow](./diagrams/01-data-flow.md) | Browser → LIQAA Cloud → LiveKit SFU |
| [02 — Auth flow (pk → JWT → JWT-on-WebSocket)](./diagrams/02-auth-flow.md) | Per-user authentication end-to-end |
| [03 — Webhook delivery](./diagrams/03-webhook-delivery.md) | Event emit → HMAC sign → exponential retry |
| [04 — WebRTC negotiation](./diagrams/04-webrtc-negotiation.md) | ICE / DTLS / SRTP between browser and SFU |
| [05 — Recording pipeline](./diagrams/05-recording-pipeline.md) | LiveKit egress → S3-compatible bucket |

### Postmortems

When we have an incident that lasts > 30 minutes, the postmortem lives here. We commit to a no-blame format and publish within 5 business days.

> No postmortems yet — but we'll publish them when we have them. The repo is configured to make it easy.

## How to contribute

Spot a flaw in our reasoning? Disagree with a decision? Open an issue or a PR. We may not change the decision, but we'll document the dissent.

```bash
adrs/NNN-short-title.md
```

Use [`adrs/template.md`](./adrs/template.md) as a starting point.

## License

[CC-BY-4.0](./LICENSE) — quote, adapt, build on this. Just give credit.
