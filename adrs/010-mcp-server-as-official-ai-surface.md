# ADR 010 — Ship an official MCP server as the AI agent surface

- **Status:** accepted
- **Date:** 2026-05-01

## Context

The Model Context Protocol (MCP) is Anthropic's open standard (announced Nov 2024) for letting LLMs talk to tools. By 2026, every serious developer tool has an MCP server: GitHub, Postgres, Linear, Figma, Stripe.

The question is not "should we?" — it's "how soon, and how official?"

## Decision

[liqaa-mcp](https://github.com/hartemyaakoub/liqaa-mcp) is published as `@liqaa/mcp`. It exposes our API as MCP tools:

- `liqaa_create_room`
- `liqaa_list_rooms`
- `liqaa_issue_token`
- `liqaa_register_webhook`
- `liqaa_query_logs`

The server is stateless, takes `LIQAA_SK` as an env var, and runs over stdio. Drop into Claude Desktop / Cursor / Continue / any MCP client.

We treat it as a **first-class supported product**, with the same SLA as our SDKs.

## Consequences

### Positive
- Developers using Claude or Cursor can *implement LIQAA via prompt*: "add a video support button to my Next.js app."
- Marketing surface — being on the [official MCP server list](https://github.com/modelcontextprotocol/servers) is a permanent backlink for our domain.
- The MCP server is a forcing function: if a tool is hard to expose to an AI agent, our API surface is probably wrong.

### Negative
- One more thing to maintain. We ship CI on every push and a publish workflow on tag.
- AI-driven usage patterns can be… creative. Rate limits and idempotency keys need to be airtight.

### Mitigations
- Same OpenAPI-derived shape as all SDKs (ADR-008).
- Heavy `Idempotency-Key` support on every mutating endpoint.

## References
- [Model Context Protocol](https://modelcontextprotocol.io)
- [Official servers list](https://github.com/modelcontextprotocol/servers)
- [Stripe MCP](https://github.com/stripe/agent-toolkit)
