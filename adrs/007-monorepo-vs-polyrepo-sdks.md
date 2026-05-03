# ADR 007 — Polyrepo for SDKs (one repo per language)

- **Status:** accepted
- **Date:** 2026-04-28

## Context

We ship SDKs for JavaScript / TypeScript, PHP, Python, Go, plus a CLI, MCP server, and templates. Two layouts:

1. **Monorepo** (`liqaa/sdks` with `js/`, `php/`, `python/`, `go/` subdirs).
2. **Polyrepo** — one GitHub repo per artifact.

Monorepos are great for *internal* code (shared types, atomic refactors). For *public* SDKs distributed via package managers, they have known pain points:

- npm / Composer / PyPI / Go modules each prefer one-package-per-repo conventions.
- Go specifically reads versioning from git tags at the *repo root* (no clean way to do `liqaa-sdks/go/v1.2.3` without abusing module paths).
- GitHub stars/topics/issues split per language are easier to navigate in separate repos.
- The "1 repo = 1 thing" rule makes our profile read like a portfolio.

## Decision

Polyrepo. One GitHub repo per SDK / tool:

- [liqaa-js](https://github.com/hartemyaakoub/liqaa-js) — npm `@liqaa/js`
- [liqaa-php](https://github.com/hartemyaakoub/liqaa-php) — Packagist `liqaa/sdk`
- [liqaa-python](https://github.com/hartemyaakoub/liqaa-python) — PyPI `liqaa`
- [liqaa-go](https://github.com/hartemyaakoub/liqaa-go) — `github.com/hartemyaakoub/liqaa-go`
- [liqaa-cli](https://github.com/hartemyaakoub/liqaa-cli) — npm `@liqaa/cli`
- [liqaa-mcp](https://github.com/hartemyaakoub/liqaa-mcp) — npm `@liqaa/mcp`
- [liqaa-openapi](https://github.com/hartemyaakoub/liqaa-openapi) — single source of truth

## Consequences

### Positive
- Zero friction with each language's package manager.
- Clean GitHub topology — devs find the SDK for *their* language without scrolling.
- Independent versioning per SDK (`@liqaa/js@1.4.0` while `liqaa/sdk@1.0.0` is fine).
- Independent CI matrices.

### Negative
- Risk of API drift between SDKs.
- Code duplication for shared concerns (webhook verification logic, retry policies).

### Mitigations
- **Single OpenAPI source** in [liqaa-openapi](https://github.com/hartemyaakoub/liqaa-openapi). All SDKs reference it.
- **Cross-repo CI**: a tag in `liqaa-openapi` triggers regeneration PRs in each SDK repo.
- **Compliance test suite** (planned) that runs the same scenario against every SDK to catch behavioural drift.

## References
- [Go modules versioning](https://go.dev/ref/mod#versions)
- [Stripe's polyrepo SDK strategy](https://github.com/stripe)
