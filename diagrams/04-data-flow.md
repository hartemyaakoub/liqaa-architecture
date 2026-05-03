# Diagram 04 — Data flow + storage

What lives where, and how long it stays.

```
                       ┌────────────────────────────┐
                       │   Customer browser         │
                       │   ─────────────────────    │
                       │   Media (DTLS-SRTP)        │
                       │   never decrypted in       │
                       │   transit                  │
                       └──────────┬─────────────────┘
                                  │
                                  ▼
              ┌──────────────────────────────────────────┐
              │   SFU (LiveKit, eu-central-1 + dz-algiers-1)│
              │   ─────────────────────────────────────  │
              │   Decrypts media (it has to, to forward) │
              │   Forwards to peers                      │
              │   If recording=on → writes to S3         │
              │   No persistent local storage            │
              └──────┬───────────────────────────────────┘
                     │ (only if recording.enabled)
                     ▼
              ┌──────────────────────────────────────────┐
              │   Object storage (Cloudflare R2)         │
              │   ─────────────────────────────────────  │
              │   Path: recordings/<projId>/<recId>.mp4  │
              │   No PII in path (ADR-005)               │
              │   Default retention: 30 days             │
              │   Pro plan: 90 days · Ent: configurable  │
              └──────────────────────────────────────────┘

              ┌──────────────────────────────────────────┐
              │   MySQL (RDS-equivalent, Frankfurt)      │
              │   ─────────────────────────────────────  │
              │   ▸ projects, api_keys                   │
              │   ▸ rooms (metadata only)                │
              │   ▸ events (90 days)                     │
              │   ▸ webhook_subscriptions, deliveries    │
              │   ▸ users, billing                       │
              └──────────────────────────────────────────┘

              ┌──────────────────────────────────────────┐
              │   Redis (queue + cache)                  │
              │   ─────────────────────────────────────  │
              │   ▸ webhook delivery queue               │
              │   ▸ rate-limit counters                  │
              │   ▸ idempotency keys (24 h)              │
              │   ▸ live participant counts (60 s TTL)   │
              └──────────────────────────────────────────┘
```

## Retention

| Data                | Default | Plan max          | Hard cap |
|---------------------|---------|-------------------|----------|
| Live media (transit)| 0 (passthrough) | — | — |
| Recordings          | 30 d    | 365 d (Ent)       | 7 years  |
| Event log           | 90 d    | 365 d (Ent)       | 365 d    |
| Webhook deliveries  | 30 d    | 90 d (Pro)        | 90 d     |
| Idempotency keys    | 24 h    | 24 h              | 24 h     |
| Logs (raw)          | 14 d    | 30 d (Pro)        | 30 d     |

On account deletion: tombstone in MySQL, hard-delete from R2 in 30 days, full purge in 90 (GDPR Art 17).
