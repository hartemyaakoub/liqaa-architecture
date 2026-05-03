# Diagram 01 — System overview

How a `pk_live_` in a customer's browser becomes a peer-to-peer call routed through our SFU.

```
                            ┌────────────────────────────┐
                            │   Customer's Browser       │
                            │   ── liqaa.io/sdk.js ──    │
                            │                            │
                            │   LIQAA.init({ publicKey,  │
                            │      sdkToken })           │
                            └──────────┬─────────────────┘
                                       │ 1. fetch sdk_token
                                       │    (POST to merchant server)
                                       ▼
                            ┌────────────────────────────┐
                            │   Merchant's Server        │
                            │   (Next.js / Laravel /…)   │
                            │   uses sk_live_ to call    │
                            │   POST /sdk-token          │
                            └──────────┬─────────────────┘
                                       │ 2. signed S2S
                                       ▼
                  ┌──────────────────────────────────────────┐
                  │   LIQAA Cloud (api.liqaa.io)             │
                  │                                          │
                  │   ▸ verify sk_live_                       │
                  │   ▸ check plan / quotas                  │
                  │   ▸ mint scoped JWT (60 min)             │
                  └──────────┬───────────────────────────────┘
                             │ 3. JWT returned
                             ▼
                  ┌──────────────────────────────────────────┐
                  │   Browser presents JWT to LiveKit signal │
                  │   wss://rtc.liqaa.io/                    │
                  └──────────┬───────────────────────────────┘
                             │ 4. ICE → TURN/STUN → media
                             ▼
              ┌────────────────────────────────────────────────┐
              │   LiveKit SFU (eu-central-1 + dz-algiers-1)    │
              │   ▸ receives 1 stream per participant          │
              │   ▸ forwards selected layers to each peer      │
              │   ▸ records to S3 (opaque paths, ADR-005)      │
              └────────────────────────────────────────────────┘
                             │
                             │ 5. lifecycle events
                             ▼
              ┌────────────────────────────────────────────────┐
              │   Webhook fan-out (HMAC-SHA256, ADR-004)        │
              │   call.started · call.ended · recording.ready  │
              │   → merchant server                            │
              └────────────────────────────────────────────────┘
```

## Latency budget

| Hop                              | Typical |
|----------------------------------|---------|
| Browser → merchant `/api/sdk-token`  | 30–80 ms |
| Merchant → LIQAA `/sdk-token`        | 20–50 ms |
| Browser → SFU (Algiers, intra-DZ)    | 5–15 ms |
| Browser → SFU (Frankfurt, EU)        | 30–60 ms |
| Browser → SFU (Frankfurt, MENA)      | 60–110 ms |
| Webhook fan-out per attempt          | 50–200 ms |
