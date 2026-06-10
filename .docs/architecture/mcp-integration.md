# Architecture: MCP Integration

This document describes how the Lore MCP Server fits into the overall Lore architecture and enables online dynamic consumption of Skills from the Registry.

## Updated Component Map

```
┌────────────────────────────────────────────────────────────────────┐
│                         AI Harnesses                               │
│                                                                    │
│  ┌────────────────┐  ┌────────────────┐  ┌─────────────────────┐  │
│  │  Claude Code   │  │    Cursor      │  │  GitHub Copilot     │  │
│  │  (MCP Client)  │  │  (MCP Client)  │  │  (Extension/REST)   │  │
│  └───────┬────────┘  └───────┬────────┘  └──────────┬──────────┘  │
│          │                   │                       │             │
│  ┌────────────────┐  ┌────────────────┐              │             │
│  │     Codex      │  │   Gemini CLI   │              │             │
│  │  (MCP Client)  │  │  (MCP Client)  │              │             │
│  └───────┬────────┘  └───────┬────────┘              │             │
└──────────┼───────────────────┼───────────────────────┼────────────┘
           │  MCP Protocol     │                        │
           │  (stdio or SSE)   │                        │ HTTP REST
           └─────────┬─────────┘                        │
                     │                                  │
         ┌───────────▼──────────────────────────────────┼──────────┐
         │              Lore MCP Server                 │          │
         │           apps/mcp-server/ (Go)              │          │
         │                                              │          │
         │  Tools:                                      │          │
         │  lore_get, lore_search, lore_resolve,        │          │
         │  lore_list, lore_versions                    │          │
         │                                              │          │
         │  Resources:                                  │          │
         │  lore://{namespace}/{name}:{version}         │          │
         │                                              │          │
         │  Renderer Layer:                             │          │
         │  claude │ cursor │ copilot │ codex │ raw     │          │
         │                                              │          │
         │  In-memory TTL cache (LORE_CACHE_TTL)        │          │
         └─────────────────────┬────────────────────────┘          │
                               │ HTTP REST                          │
                               │                                    │
         ┌─────────────────────▼────────────────────────────────────▼──┐
         │                    Lore Registry API                         │
         │                   apps/registry/ (Go)                        │
         │                                                              │
         │  Content API:  GET /v1/content/{ns}/{name}/{ver}             │
         │                GET /v1/content/{ns}/{name}/{ver}/body        │
         │  Package API:  GET /v1/packages/{ns}/{name}/{ver}            │
         │                GET /v1/packages/{ns}/{name}/versions         │
         │                GET /v1/packages/{ns}/{name}/{ver}/raw        │
         │  Search API:   GET /v1/search                                │
         │  Info:         GET /_lore/info                               │
         │  Write API:    POST/DELETE /v1/packages (auth required)      │
         │  Namespace:    POST/PATCH /v1/namespaces (auth required)     │
         │                                                              │
         │  Auth: stateless OIDC JWT (ADR-0005)                         │
         │  RBAC: namespace.yaml embedded roles (ADR-0006)              │
         └──────────────────────────┬───────────────────────────────────┘
                                    │
         ┌──────────────────────────▼───────────────────────────────────┐
         │                      Object Storage                           │
         │          S3 / Azure Blob Storage / MinIO / Local FS           │
         │                                                               │
         │  m1cloud/namespace.yaml                                       │
         │  m1cloud/packages/azure-architect-1.0.0.lore                 │
         │  m1cloud/cache/azure-architect-1.0.0/en-US.md  ← extracted   │
         │  m1cloud/cache/azure-architect-1.0.0/pt-BR.md  ← extracted   │
         │  m1cloud/cache/azure-architect-1.0.0/manifest.json           │
         └───────────────────────────────────────────────────────────────┘
```

## Deployment Modes

### Developer local (stdio)

The developer installs `lore-mcp` and configures their AI harness to spawn it as a subprocess. The binary communicates via stdin/stdout. No network port is opened.

```
AI Harness ──stdio──► lore-mcp ──HTTPS──► registry.lore.io
```

### Self-hosted (HTTP/SSE)

Teams run `lore-mcp` as a container alongside their private Registry instance. AI harnesses connect over the local network.

```
AI Harnesses ──SSE──► lore-mcp:8080 ──HTTP──► registry:8080
```

### Public hosted service (mcp.lore.io)

Anthropic / Lore team hosts `lore-mcp` as a public service. AI harnesses connect over HTTPS without installing any binary. Authentication uses bearer tokens for private namespaces.

```
AI Harnesses ──HTTPS/SSE──► mcp.lore.io ──HTTPS──► registry.lore.io
```

## Data Flow: `lore_get` Call

```
1. User asks AI to use "azure-architect" skill
2. AI calls tool: lore_get({"id": "m1cloud/azure-architect:latest"})
3. MCP Server resolves "latest" → calls GET /v1/packages/m1cloud/azure-architect/versions
4. MCP Server calls GET /v1/content/m1cloud/azure-architect:1.0.0/body?lang=en-US
5. Registry checks content cache:
   a. Cache HIT: serves m1cloud/cache/azure-architect-1.0.0/en-US.md from storage
   b. Cache MISS: fetches azure-architect-1.0.0.lore, extracts, writes to cache, serves
6. MCP Server applies target renderer (e.g., claude → prepends HTML comment)
7. MCP Server writes result to in-memory TTL cache
8. MCP Server returns rendered markdown as tool result
9. AI uses content as context for current session
```

## What Is Not Stored on the Developer's Machine

Unlike `lore pull`, the MCP path writes nothing to disk on the developer's machine. The Skill content exists only in the AI's context window for the duration of the session. This is the intended behavior for the online dynamic use case.

## Relationship to the CLI

The CLI (`apps/cli/`) and the MCP Server (`apps/mcp-server/`) serve complementary use cases:

| | CLI (`lore pull`) | MCP Server (`lore_get`) |
|--|--|--|
| Writes files to disk | Yes | No |
| Platform-specific artifacts | Yes (SKILL.md, MDC, etc.) | No (markdown in context only) |
| Works offline | Yes (after pull) | No |
| Suitable for | Project setup, CI, committed config | Dynamic sessions, agents, exploration |
| Auth | `lore login` (device grant) | `LORE_AUTH_TOKEN` env var |

Both components share the same rendering logic via `apps/pkg/renderer/` (a shared Go library), ensuring consistent output between install-time and online rendering.
