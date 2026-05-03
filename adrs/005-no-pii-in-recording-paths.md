# ADR 005 — No PII in recording filenames or storage paths

- **Status:** accepted
- **Date:** 2026-04-27

## Context

When we record a call, we store the file. Naïvely, you might use:

```
recordings/customer@example.com/2026-05-03_support-call.mp4
```

This is a privacy disaster:

1. The S3 bucket name + path is logged everywhere — load balancers, CDN access logs, server logs, BI tooling.
2. If the bucket leaks (or a stale CDN cache returns 200 for the wrong path), the customer's email is exposed.
3. GDPR Article 32 asks for "appropriate technical and organisational measures" — putting PII in URLs is the opposite of that.

## Decision

Storage paths are **opaque random IDs**, never PII:

```
recordings/<projectId>/<recordingId>.mp4
recordings/proj_01HK7MWQ3X/rec_01HK7N0DEA.mp4
```

Mappings between `recordingId` and the call's participants live in our database, behind authentication. Pre-signed URLs (15 min expiry) give merchants temporary access.

We hash any user-identifying data we *must* log (e.g., participant ID in distributed traces) with a per-project salt.

## Consequences

### Positive
- A leak of an S3 path tells you nothing about the customer.
- Compliance with GDPR / Algeria Law 18-07 / CCPA "data minimisation".
- Easier to delete: one `rec_…` row → one S3 object.

### Negative
- Slightly harder to debug ("which recording is this?"). We expose the mapping in the Console UI for authenticated merchants.

## References
- [GDPR Article 32](https://gdpr-info.eu/art-32-gdpr/)
- Algeria Loi n° 18-07 du 10 juin 2018, art. 5–6
