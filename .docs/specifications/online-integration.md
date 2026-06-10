# Online Integration Guide

This document describes how each AI runtime connects to the Lore Registry dynamically, without local file installation. The primary mechanism is the **Lore MCP Server** (`lore-mcp`). For runtimes without MCP support, the Registry REST API is used directly.

For the MCP Server tool/resource spec, see [mcp-server.md](./mcp-server.md).
For the Registry content endpoints, see [content-api.md](./content-api.md).

---

## How Online Integration Works

```
AI Harness  ──MCP──►  lore-mcp  ──HTTP──►  Registry API  ──►  Storage
                      (renders)             (extracts)
```

1. The AI calls a `lore_*` tool or reads a `lore://` resource
2. The MCP Server fetches content from the Registry via `/v1/content/`
3. The Registry extracts the relevant content from the `.lore` archive (cached after first access)
4. The MCP Server applies a target-specific renderer (minimal markdown transform)
5. The AI receives the rendered markdown and uses it as context for the current session

The content is **ephemeral** — it lives only in the AI's context window for the session. Nothing is written to disk on the developer's machine. For persistent local installation, use `lore pull` (the CLI).

---

## Claude Code

Claude Code supports MCP natively via `.claude/settings.json`.

### Setup

**Option A — local binary (stdio):**

Install `lore-mcp` (single binary, no runtime dependencies), then add to `.claude/settings.json`:

```json
{
  "mcpServers": {
    "lore": {
      "command": "lore-mcp",
      "env": {
        "LORE_REGISTRY_URL": "https://registry.lore.io",
        "LORE_TARGET": "claude",
        "LORE_DEFAULT_LANG": "en-US"
      }
    }
  }
}
```

**Option B — hosted service (HTTP/SSE):**

No local binary needed. Claude connects to `mcp.lore.io`:

```json
{
  "mcpServers": {
    "lore": {
      "type": "sse",
      "url": "https://mcp.lore.io/sse",
      "headers": {
        "Authorization": "Bearer <your-lore-token>"
      }
    }
  }
}
```

The `Authorization` header is only required for private namespaces. Public packages work without it.

### Usage

Claude will automatically discover `lore_search`, `lore_get`, `lore_resolve`, `lore_list`, and `lore_versions` as available tools.

Example interactions:

```
User: Use the azure-architect skill from the Lore Registry.
Claude: [calls lore_get({"id": "m1cloud/azure-architect:latest"})]
        [receives markdown content]
        [applies it as context for the rest of the conversation]

User: Find me skills related to Terraform on Azure.
Claude: [calls lore_search({"query": "terraform azure", "kind": "Skill"})]
        [returns list of matching packages]

User: I need to design an AKS cluster with best practices.
Claude: [calls lore_resolve({"goal": "design AKS cluster with best practices"})]
        [fetches and returns top relevant packages with full content]
```

Claude can also read packages as MCP resources directly by URI:

```
Read lore://m1cloud/azure-architect:1.0.0
```

### Target rendering for Claude

The `claude` renderer prepends an HTML comment to the markdown:

```html
<!-- lore:Skill:m1cloud/azure-architect:1.0.0 -->
# Azure Architect
...
```

This is invisible in rendered markdown but allows Claude to track which packages are in context and reference them by identifier.

---

## Cursor

Cursor supports MCP via `.cursor/mcp.json` at the project or user level.

### Setup

```json
{
  "mcpServers": {
    "lore": {
      "command": "lore-mcp",
      "args": [],
      "env": {
        "LORE_REGISTRY_URL": "https://registry.lore.io",
        "LORE_TARGET": "cursor",
        "LORE_DEFAULT_LANG": "en-US"
      }
    }
  }
}
```

For auto-detection to work without setting `LORE_TARGET`, Cursor must send `clientInfo.name = "cursor"` during MCP initialization (behavior depends on Cursor's MCP implementation version).

### Usage

Same tools as Claude Code. The `cursor` renderer prepends MDC-compatible frontmatter:

```
---
description: Azure Architect skill from Lore Registry (m1cloud/azure-architect:1.0.0)
alwaysApply: false
---

# Azure Architect
You are an expert Azure solutions architect...
```

This format is compatible with Cursor's MDC rule system. The content is injected into context for the current conversation, not written to `.cursor/rules/`. For persistent MDC rules, use `lore pull --target cursor`.

---

## GitHub Copilot

GitHub Copilot does not currently support MCP natively. Integration is via **GitHub Copilot Extensions** (REST webhook model).

### Setup

A Copilot Extension handler calls the Registry Content API directly to inject Skills into Copilot's context:

```
GET https://registry.lore.io/v1/content/m1cloud/azure-architect:latest/body?lang=en-US
```

The response markdown is injected as a `system` or `context` message in the Copilot conversation payload according to the Copilot Extensions spec.

**No MCP Server is needed for this path.** The Copilot Extension itself acts as the adapter.

### Future MCP support

When GitHub Copilot adds native MCP support, use the same `lore-mcp` binary with `LORE_TARGET=copilot`. The `copilot` renderer currently returns plain markdown (no transform needed).

---

## Codex (OpenAI CLI)

Codex CLI supports MCP via `codex.yaml` at the project level or `~/.codex/config.toml` for user-level config.

### Setup (`codex.yaml`)

```yaml
mcp_servers:
  lore:
    command: lore-mcp
    env:
      LORE_REGISTRY_URL: https://registry.lore.io
      LORE_TARGET: codex
      LORE_DEFAULT_LANG: en-US
```

### Setup (`~/.codex/config.toml`)

```toml
[mcp_servers.lore]
command = "lore-mcp"
env = { LORE_REGISTRY_URL = "https://registry.lore.io", LORE_TARGET = "codex" }
```

The `codex` renderer returns plain markdown with no transform.

---

## Gemini CLI

Gemini CLI supports MCP via `~/.gemini/settings.json` or a project-level equivalent.

### Setup

```json
{
  "mcpServers": [
    {
      "name": "lore",
      "command": "lore-mcp",
      "env": {
        "LORE_REGISTRY_URL": "https://registry.lore.io",
        "LORE_TARGET": "gemini",
        "LORE_DEFAULT_LANG": "en-US"
      }
    }
  ]
}
```

The `gemini` renderer returns plain markdown with no transform.

---

## Private Namespaces

For private namespaces, the MCP Server must present a valid OIDC access token to the Registry.

**Option A — static service token (CI/CD, team setup):**

```env
LORE_AUTH_TOKEN=<personal-access-token-or-service-account-token>
```

**Option B — per-call token passthrough (future):**

The `lore_get` and `lore_search` tools will accept an optional `auth_token` parameter in a future version, allowing the AI to present a token obtained via `lore login` for the current session.

For now, team setups should configure `LORE_AUTH_TOKEN` at the server level.

---

## Comparison: Online vs. Install-time

| | Online (MCP) | Install-time (CLI) |
|--|--|--|
| **Mechanism** | MCP tools / REST API | `lore pull` |
| **Storage** | AI context window only | Files on disk |
| **Persistence** | Session only | Until removed |
| **Platform files** | No (AI reads markdown directly) | Yes (`SKILL.md`, MDC rules, etc.) |
| **Suitable for** | Dynamic, exploratory, agent workflows | Project setup, committed config |
| **Works offline** | No | Yes (after pull) |
| **Auth for private** | `LORE_AUTH_TOKEN` in MCP Server | `lore login` |
