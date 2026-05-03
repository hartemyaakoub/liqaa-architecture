# Diagram 05 — Multi-region failover

What happens when Algiers loses uplink at 14:30.

```
Normal operation (both regions healthy)
─────────────────────────────────────────

   DZ user ─── 8 ms ───▶ algiers-1 SFU  ──┐
                                          │  ── replication of metadata
   EU user ─── 30 ms ──▶ frankfurt-1 SFU ─┘     (no media, only state)


Algiers goes dark
─────────────────────────────────────────

   1. Cloudflare DNS LB health check fails (3× 60s = 3 min detection)
   2. DZ users now resolve api.liqaa.io → frankfurt-1 IP
   3. SDK reconnects, latency jumps 8 ms → 60 ms (still functional)
   4. Active calls in algiers-1 drop. SDK auto-reconnect (3× retry, 1s/2s/5s)
      → clients land in frankfurt-1 within ~10 seconds
   5. Status page (liqaa-status) opens an incident issue automatically

   DZ user ─── 60 ms ──▶ frankfurt-1 SFU
   EU user ─── 30 ms ──▶ frankfurt-1 SFU


Algiers recovers
─────────────────────────────────────────

   1. Probes return green for 3 consecutive checks (15 min)
   2. DNS LB re-introduces algiers-1 to the pool
   3. New connections from DZ go to algiers-1 (latency drops back to 8 ms)
   4. Existing calls stay on frankfurt-1 until they end (no forced rollback)
   5. Incident issue closed with timeline + RCA
```

## Failure modes we explicitly handle

| Failure                          | Detection | Mitigation                                                          |
|----------------------------------|-----------|---------------------------------------------------------------------|
| Algiers uplink down              | 3 min     | DNS failover → frankfurt-1                                           |
| Frankfurt network split          | 3 min     | All traffic to algiers-1; admin / dashboard degraded for non-DZ     |
| Single SFU pod OOM               | 30 s      | k8s liveness restart; LB drains the pod; clients reconnect          |
| Database primary down            | 60 s      | Automatic promotion of replica (RDS-equivalent failover)             |
| R2 bucket region down            | 5 min     | Recordings queued in Redis; flushed when bucket returns              |
| DNS provider outage              | n/a       | Secondary NS configured (manual cutover, ~15 min RTO)                |

## What we don't handle (yet)

- **Multi-region active-active for live media**: requires SFU mesh peering. Roadmap Q4 2026.
- **Cross-region recording reconciliation**: a recording started in algiers-1 isn't continued in frankfurt-1 if the SFU dies mid-call. The recording for that call ends.
