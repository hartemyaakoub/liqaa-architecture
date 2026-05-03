# Diagram 03 — Webhook delivery + retry

How `call.ended` reaches the merchant, with HMAC + 5-attempt backoff.

```
SFU emits event              Webhook gateway              Merchant /webhooks
       │                            │                            │
       │  internal event bus        │                            │
       │ ─────────────────────────▶│                            │
       │                            │                            │
       │                            │ 1. find subscriptions      │
       │                            │    for project + event     │
       │                            │                            │
       │                            │ 2. build payload           │
       │                            │    sign:                   │
       │                            │      ts = now              │
       │                            │      v1 = HMAC-SHA256(     │
       │                            │        secret,             │
       │                            │        ts + "." + body)    │
       │                            │                            │
       │                            │ 3. POST                    │
       │                            │    LIQAA-Signature:        │
       │                            │      t=…,v1=…              │
       │                            │ ─────────────────────────▶│
       │                            │                            │
       │                            │                            │ 4. verify
       │                            │                            │    constant-time
       │                            │                            │    compare
       │                            │                            │
       │                            │  ◀─────  2xx  ──────────  │
       │                            │                            │
       │                            │ 5. mark delivered          │
```

## Retry policy

```
attempt   wait        cumulative
   1      0            0
   2      1 min        1 min
   3      5 min        6 min
   4      30 min       36 min
   5      2 h          2h 36 min
   6      12 h         14h 36 min   ← last attempt, then dead-lettered
```

A delivery is dead-lettered after 5 failures. The merchant can replay from the Console UI's webhook log for 30 days.

## What we send

```json
{
  "id": "evt_01HK7N0DEA",
  "type": "call.ended",
  "created": 1714680123,
  "data": {
    "room": "support",
    "duration": 184,
    "participants": ["agent_07", "user_b9q"],
    "recording_url": null
  }
}
```

Headers:

```
Content-Type: application/json
LIQAA-Signature: t=1714680123,v1=8a4f…c721
LIQAA-Event-Id: evt_01HK7N0DEA
LIQAA-Delivery-Attempt: 1
```
