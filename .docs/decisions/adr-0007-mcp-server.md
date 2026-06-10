# ADR-0007: Introduce Lore MCP Server as a Separate Component

**Status:** Accepted

**Date:** 2026-06-10

---

## Context

AI harnesses that support the Model Context Protocol (MCP) — Claude Code, Cursor, Codex, Gemini CLI — need a server-side counterpart to fetch Lore packages dynamically without installing files locally. The Registry API already serves packages, but it is not an MCP server; it speaks HTTP REST, not the JSON-RPC 2.0 MCP protocol.

Two options were considered:

1. **Embed MCP handling in the Registry API.** The Registry listens on an additional MCP/SSE endpoint alongside its REST endpoints.
2. **Introduce a separate `lore-mcp` binary** that speaks MCP and proxies content requests to the Registry API.

---

## Decision

Introduce `apps/mcp-server/` as an independent Go binary (`lore-mcp`) that speaks MCP and delegates all data fetching to the Registry API via HTTP.

---

## Consequences

**Benefits:**

- **Deployment flexibility.** `lore-mcp` can be run in three modes independently: as a local process spawned by the AI harness (stdio), as a self-hosted container, or as a public hosted service at `mcp.lore.io`. Embedding MCP in the Registry would couple its deployment mode to what the Registry needs.
- **Registry stays focused.** The Registry API is a REST service for package storage, validation, and distribution. MCP is a presentation protocol for AI harnesses. Keeping them separate avoids adding AI-specific protocol logic to the Registry codebase.
- **MCP protocol evolution.** The MCP spec is still evolving. Isolating it in `lore-mcp` means MCP changes (new transports, new capability negotiation) don't affect the Registry.
- **Independent versioning.** `lore-mcp` and the Registry can be versioned and released independently. A breaking MCP spec update does not require a Registry deployment.
- **Local binary model.** Developers can `brew install lore-mcp` or download the binary and run it locally without standing up a Registry instance.

**Trade-offs:**

- **One more binary to maintain.** The ecosystem grows: CLI, Registry, MCP Server. Each needs its own release pipeline and versioning.
- **Extra network hop** for locally-run setups: AI → lore-mcp (stdio) → Registry API (HTTP). In practice this is a localhost or LAN call and adds negligible latency.
- **Content duplication in memory.** `lore-mcp` caches content in memory. The Registry also caches extracted content in storage. Two layers of caching exist, but they serve different purposes (Registry: durable extraction cache; MCP Server: session-scoped latency cache).

The benefits significantly outweigh the trade-offs for the expected deployment patterns of this system.
