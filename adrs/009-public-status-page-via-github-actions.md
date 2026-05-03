# ADR 009 — Public status page powered by GitHub Actions probes

- **Status:** accepted
- **Date:** 2026-04-30

## Context

A serious developer-facing platform needs a public status page. Customers want to know:
- "Is the API up *right now*?"
- "Was there an outage when my call dropped yesterday?"
- "What's the historical uptime?"

Buying StatusPage.io ($79–$1500/month) for an early-stage product is overkill, but **silence after an incident is worse than a bug**.

## Decision

[liqaa-status](https://github.com/hartemyaakoub/liqaa-status) runs probes via GitHub Actions every 5 minutes. Each probe:

- Hits 6 critical endpoints (`/health`, `/api/v1/ping`, edge SFU TURN, signaling WS, dashboard, docs).
- Records latency + status to `results/YYYY-MM/DATE_TIME.json`.
- Updates `latest.json` (machine-readable for [liqaa.io/status](https://liqaa.io/status) widget).
- Commits results back to the repo, so history is **a public git log**.

When a probe fails 2 times in a row, an issue is auto-opened in the repo (`incident:` label) and the LIQAA team is paged.

## Consequences

### Positive
- **Zero monthly cost** (within GitHub Actions free tier on a public repo).
- **Fully transparent** — incident logs are git history, not hidden in someone's dashboard.
- **Customers can subscribe** by watching the repo (releases + issues).
- **Independent of our infra** — if our cloud goes down, the probes (running on Microsoft's infra) still report the truth.

### Negative
- 5-minute resolution. Real outages can be missed if they last < 5 min.
- GitHub Actions occasionally has its own latency (~30s queue time during peak hours).
- Public history of every blip might look bad. We accept that — honesty > polish.

### Mitigations
- For sub-5-minute resolution we'll add a separate paid Pingdom in front of the SFU when MRR > $5k/month.
- Probes use multiple GitHub Action regions to reduce false positives from a single CI runner.

## References
- [liqaa-status](https://github.com/hartemyaakoub/liqaa-status)
- [GitHub Actions free tier](https://docs.github.com/en/actions/learn-github-actions/usage-limits-billing-and-administration)
