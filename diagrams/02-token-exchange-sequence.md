# Diagram 02 — Token exchange sequence

Stripe-style: `pk_live_` in browser, `sk_live_` on server, JWT in between.

```
Browser              Merchant server       LIQAA API           LiveKit SFU
   │                       │                    │                    │
   │  1. user clicks ▶︎    │                    │                    │
   │ ──────────────────▶  │                    │                    │
   │                       │                    │                    │
   │                       │  2. POST /sdk-token                    │
   │                       │     Authorization: sk_live_…           │
   │                       │     { user, room, ttl }                │
   │                       │ ─────────────────▶│                    │
   │                       │                    │                    │
   │                       │                    │ 3. mint JWT (HS256)│
   │                       │                    │    sub, room, exp  │
   │                       │ ◀─────────────────│                    │
   │                       │     { jwt }        │                    │
   │  4. { jwt }           │                    │                    │
   │ ◀──────────────────  │                    │                    │
   │                       │                    │                    │
   │  5. wss connect with jwt                                        │
   │ ─────────────────────────────────────────────────────────────▶│
   │                       │                    │                    │
   │                       │                    │    6. verify HS256 │
   │                       │                    │    ◀──── shared ───│
   │                       │                    │           secret   │
   │                       │                    │                    │
   │  7. admitted, media flows (DTLS-SRTP)                            │
   │ ◀────────────────────────────────────────────────────────────▶│
```

## Why two steps?

If we let the browser hit `/sdk-token` directly with `pk_live_`, anyone could mint tokens for any user identity — there's no server verifying *who* the browser actually is.

The merchant server is the only thing that knows:
- The signed-in user's email / id
- Whether they're authorised to join this room

So **the merchant server is the issuance authority**. LIQAA mints, the merchant authorises.
