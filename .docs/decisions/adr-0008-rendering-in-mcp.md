# ADR-0008: Content Rendering Happens in the MCP Server, Not in the Registry API

**Status:** Accepted

**Date:** 2026-06-10

---

## Context

Different AI runtimes expect content in slightly different formats:
- Cursor expects MDC-compatible YAML frontmatter
- Claude Code benefits from HTML comment metadata annotations
- GitHub Copilot, Codex, and Gemini work well with plain markdown

When a consumer calls `lore_get` on the MCP Server, it needs to receive content in the format appropriate for its runtime. The question is: where does this transformation happen?

Three options were evaluated:

**Option A — Registry API renders per request.** Add a `?target=claude` query parameter to `/v1/content/`. The Registry learns about AI platform formats and transforms the markdown before returning it.

**Option B — MCP Server renders, Registry returns neutral markdown.** The Registry always returns raw markdown. The MCP Server has a rendering layer that applies target-specific transforms based on the active target.

**Option C — MCP Server auto-detects and renders, no explicit parameter needed.** Same as Option B, but the MCP Server detects the target automatically from MCP `clientInfo.name` during initialization.

---

## Decision

Option B as the primary mechanism, with Option C layered on top as auto-detection convenience.

The Registry API returns neutral markdown. The MCP Server is responsible for all target-specific rendering.

---

## Consequences

**Benefits:**

- **ADR-0004 is preserved at the API boundary.** ADR-0004 states that Lore packages must be runtime-agnostic. This principle extends naturally to the Registry API: the Registry returns content that is independent of any AI platform. Platform-specific knowledge lives only in the MCP Server's renderer layer.
- **Adding a new runtime requires no Registry change.** When a new AI platform needs a specific format, a new renderer is added to `apps/mcp-server/internal/renderer/`. The Registry is not touched.
- **Simpler caching.** The Registry caches extracted content at a single canonical representation. If the Registry rendered per-target, the same `{namespace}/{name}/{version}` would have N cache entries (one per target), multiplying storage and cache complexity.
- **URL semantics are clean.** `/v1/content/m1cloud/azure-architect:1.0.0` always returns the same response regardless of who calls it. This is the correct REST behavior and enables HTTP-level caching (CDN, proxy) without Vary headers per target.
- **Copilot Extension path is unaffected.** GitHub Copilot Extensions call the Registry Content API directly (no MCP). These callers want neutral markdown and inject it themselves. Option A would have forced them to pass `?target=raw` defensively.

**Trade-offs:**

- **Rendering logic is not centralized.** If two consumers (e.g., the CLI's render command and the MCP Server) both render for `cursor`, they each carry their own renderer implementation. Mitigation: extract renderers into a shared Go library at `apps/pkg/renderer/` that both the CLI and MCP Server import.
- **Target detection can fail.** Auto-detection from `clientInfo.name` is not standardized; different MCP client implementations may not send consistent values. Mitigation: `LORE_TARGET` env var provides a reliable explicit override; `raw` is the safe default when detection fails.
