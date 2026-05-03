# ADR 006 — Self-hosted edge nodes (Algeria + Frankfurt) instead of global CDN-only

- **Status:** accepted
- **Date:** 2026-04-28

## Context

Real-time WebRTC media is latency-sensitive. Users in Algeria connecting to a media server in Virginia experience ~140ms RTT — every additional hop is audible jitter.

Generic CDNs (Cloudflare, Fastly) are great for HTTP and even for **playback** of recorded video, but they don't run SFUs. We must run our own.

Options:
1. Single region (Frankfurt) — covers EU + MENA at 30–80ms.
2. Two regions (Frankfurt + Algiers) — Algeria at 5–15ms, EU at 30–60ms.
3. Five+ regions globally (US-East, US-West, EU-West, ASEAN, MENA) — needed only when we have customers far from EU.

## Decision

Start with **two regions: Frankfurt (eu-central-1) + Algiers (on-prem datacentre via Djezzy peering)**.

- Frankfurt is the default. Hetzner AX-line bare metal, dedicated 10 Gbit, ~€60/server/month.
- Algiers is the latency-killer for Algerian merchants. Routes via local IXP, p99 RTT < 15 ms inside Algeria.

Clients pick the closest region via DNS-based GeoIP routing (via Cloudflare DNS load balancing).

## Consequences

### Positive
- 10x lower latency for our primary market vs going through EU.
- Sovereignty narrative: "your customer data stays in Algeria" is a real selling point for telcos and gov.
- Cost predictable (bare metal beats hyperscaler bandwidth fees by ~5x).

### Negative
- Two regions = two ops surfaces. We've automated config via Ansible to keep them in sync.
- Algiers datacentre power/uplink is a single point of failure. Frankfurt is the failover (clients auto-fall-back).
- We must monitor **per-region** SLOs, not global.

### Mitigations
- The probes in [liqaa-status](https://github.com/hartemyaakoub/liqaa-status) hit each region separately.
- Frankfurt is sized to handle 100% of traffic if Algiers goes down (with degraded latency).

## References
- [Hetzner dedicated servers](https://www.hetzner.com/dedicated-rootserver/)
- [Cloudflare DNS load balancing](https://developers.cloudflare.com/load-balancing/)
