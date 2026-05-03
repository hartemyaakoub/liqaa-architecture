# ADR 001 — Use LiveKit SFU instead of building our own

- **Status:** accepted
- **Date:** 2026-04-25
- **Decision-makers:** Yaakoub HARTEM
- **Supersedes:** —

## Context

A video conferencing API needs to route media between participants. For 2 peers, browser-native WebRTC peer-to-peer is enough. For 3+ peers, you need a Selective Forwarding Unit (SFU) — a server that receives one stream from each participant and forwards copies to the others.

We had three options:

1. **Build our own SFU** in Go/Rust on top of [Pion](https://github.com/pion/webrtc) or [mediasoup](https://mediasoup.org/).
2. **Use a managed SFU API** like [Daily.co](https://daily.co), [Twilio Programmable Video](https://www.twilio.com/video), or [Agora](https://agora.io).
3. **Use an open-source SFU** we self-host: [LiveKit](https://livekit.io), [Janus](https://janus.conf.meetecho.com/), [Jitsi](https://jitsi.org/).

## Decision

Adopt **LiveKit SFU**, self-hosted on our infrastructure.

## Consequences

### Positive

- **Production-ready from day one.** LiveKit is already battle-tested at scale (Discord, OpenAI's voice mode, Reddit). Building an SFU is a 2–3 year engineering project. We don't have 3 years.
- **Simulcast + adaptive bitrate built-in.** Critical for users on flaky mobile connections (the majority of our target market in MENA).
- **Open-source (Apache 2.0).** No vendor lock-in. If LiveKit Inc. disappears, we keep running.
- **Active community.** ~10K GitHub stars, dozens of contributors. Issues get fixed.
- **Self-hostable for enterprise.** Some customers (telcos, gov) require on-prem. LiveKit ships a Helm chart.
- **Cost.** ~10x cheaper than Twilio/Daily at scale (we pay only for compute).

### Negative

- **We must operate the SFU.** That means monitoring, scaling, version upgrades, kernel-level tuning. We absorb that operational burden.
- **No gRPC media APIs (yet).** LiveKit's API is HTTP + WebSocket. Streaming gRPC for fanout-to-many would be nicer for some use cases.
- **Recording is egress-based.** Composite recording requires running an extra `livekit-egress` service.

### Mitigations

- We've packaged the LiveKit SFU + signaling + Postgres + Redis as a single `docker-compose` stack. Self-hosters get one-command deployment.
- We monitor SFU CPU + memory via Prometheus → Grafana. Auto-scaling triggers at 70% CPU.
- We've contributed minor patches upstream (TBD: link PRs once merged).

## Alternatives considered

### Build our own (rejected)

We'd need ~6–12 months to reach feature parity with LiveKit, and another 12 months to reach production stability. The opportunity cost is enormous. Even if we succeeded, we'd be solving a solved problem — there's no differentiation in "yet another SFU".

### Managed (rejected)

Twilio: $0.004/min × thousands of users = unsustainable margins on a freemium plan. Daily: same.

We need control over our cost structure.

### Janus / Jitsi (rejected)

Both are excellent, but their developer experience is markedly worse than LiveKit's. We optimized for the team velocity of a small startup.

## References

- [LiveKit GitHub](https://github.com/livekit/livekit)
- [Comparison: LiveKit vs mediasoup vs Janus](https://blog.livekit.io/livekit-vs-mediasoup-vs-janus/)
- Internal benchmark: see [liqaa-benchmarks](https://github.com/hartemyaakoub/liqaa-benchmarks)
