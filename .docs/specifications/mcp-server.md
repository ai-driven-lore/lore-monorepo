# Lore MCP Server Specification

The Lore MCP Server (`apps/mcp-server/`) is a standalone Go binary that implements the [Model Context Protocol](https://modelcontextprotocol.io) and acts as an adapter between AI harnesses and the Lore Registry API. It allows any MCP-capable AI (Claude Code, Cursor, Gemini CLI, Codex) to discover and consume Skills, Rules, Specs, and Templates from the Registry dynamically — without installing files locally.

For local file installation, see the CLI spec. The MCP Server is the online dynamic path.

---

## Transports

The server supports two transport modes:

| Mode | Use case | How to activate |
|------|----------|-----------------|
| `stdio` | Local developer setup; AI harness spawns the process | Default; no flag needed |
| `http-sse` | Hosted service (`mcp.lore.io`); AI connects over network | `--transport sse --port 8080` |

---

## Configuration

All configuration via environment variables (12-factor). A `--config` flag may point to a YAML file with the same keys in snake_case.

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `LORE_REGISTRY_URL` | Yes | — | Base URL of the Registry API (e.g., `https://registry.lore.io`) |
| `LORE_AUTH_TOKEN` | No | — | Bearer token for accessing private namespaces |
| `LORE_TARGET` | No | `raw` | Default rendering target when not auto-detected or overridden per call |
| `LORE_DEFAULT_LANG` | No | `en-US` | Default content language when not specified in tool call |
| `LORE_CACHE_TTL` | No | `300` | Seconds to cache fetched content in memory. `0` disables cache. |

---

## Target Detection

The server determines which rendering target to use in the following priority order:

1. `target` parameter in the tool call (explicit, per-call override)
2. `LORE_TARGET` environment variable (process-level default)
3. Auto-detection from MCP initialization `clientInfo.name`:

| `clientInfo.name` value | Resolved target |
|-------------------------|-----------------|
| `claude-code` | `claude` |
| `cursor` | `cursor` |
| `github-copilot` | `copilot` |
| `codex` | `codex` |
| `gemini` | `gemini` |
| *(any other value)* | `raw` |

4. Default: `raw` (markdown with no platform-specific wrapper)

The `clientInfo.name` mapping is user-configurable in the server's YAML config under `target_map`.

---

## Tools

### `lore_get`

Retrieves the full content of a Lore package, rendered for the active target runtime. Dependencies are inlined when requested.

**Input schema:**

```json
{
  "type": "object",
  "required": ["id"],
  "properties": {
    "id": {
      "type": "string",
      "description": "Package identifier: 'namespace/name:version'. Version accepts 'latest', full semver, or semver ranges ('^1.0', '~1.0.0').",
      "examples": ["m1cloud/azure-architect:latest", "m1cloud/azure-architect:1.0.0"]
    },
    "lang": {
      "type": "string",
      "description": "BCP 47 language tag. Defaults to server LORE_DEFAULT_LANG.",
      "examples": ["en-US", "pt-BR"]
    },
    "include_deps": {
      "type": "boolean",
      "default": true,
      "description": "When true, resolved dependency content is appended after the main content."
    },
    "target": {
      "type": "string",
      "enum": ["claude", "cursor", "copilot", "codex", "gemini", "raw"],
      "description": "Rendering target override. Defaults to server target detection."
    }
  }
}
```

**Response format (text content):**

```
[Lore Registry — m1cloud/azure-architect:1.0.0 — Skill]
Source: https://registry.lore.io/v1/content/m1cloud/azure-architect:1.0.0
Language: en-US
Dependencies: m1cloud/terraform-reviewer:1.1.0 (included below)

---

# Azure Architect

You are an expert Azure solutions architect...

[full markdown content of the package]

---
[Dependency: m1cloud/terraform-reviewer:1.1.0]

# Terraform Reviewer

...
```

The header block provides attribution and version metadata so the AI can answer questions like "which version of azure-architect are you using?".

---

### `lore_search`

Searches the Registry for packages matching a query.

**Input schema:**

```json
{
  "type": "object",
  "required": ["query"],
  "properties": {
    "query": {
      "type": "string",
      "description": "Search terms. Matches title, description, and tags."
    },
    "kind": {
      "type": "string",
      "enum": ["Skill", "Rule", "Spec", "Template"],
      "description": "Filter by package type."
    },
    "namespace": {
      "type": "string",
      "description": "Restrict to a specific namespace."
    },
    "tags": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Filter by tags (all must match)."
    },
    "limit": {
      "type": "integer",
      "default": 10,
      "maximum": 50
    }
  }
}
```

**Response format (text content):**

```
Found 3 packages matching "azure terraform":

1. m1cloud/azure-architect:1.0.0 [Skill]
   Azure architecture specialist
   Tags: azure, architecture | Languages: en-US, pt-BR

2. m1cloud/terraform-reviewer:1.1.0 [Skill]
   Terraform code review specialist
   Tags: terraform, iac | Languages: en-US

3. community/azure-landing-zone:2.0.0 [Spec]
   Azure Landing Zone implementation spec
   Tags: azure, landing-zone | Languages: en-US
```

---

### `lore_resolve`

Given a natural language goal description, finds and returns the most relevant packages with their content pre-fetched. This is the AI-assisted discovery entry point — the AI does not need to know package names in advance.

**Input schema:**

```json
{
  "type": "object",
  "required": ["goal"],
  "properties": {
    "goal": {
      "type": "string",
      "description": "Natural language description of the task or problem.",
      "examples": [
        "I need to design a Terraform module for Azure AKS",
        "enforce Python naming conventions in this project"
      ]
    },
    "lang": {
      "type": "string",
      "description": "Preferred content language."
    },
    "max_results": {
      "type": "integer",
      "default": 3,
      "maximum": 5
    }
  }
}
```

**Response:** same format as `lore_get` for the top result, with summaries of additional results appended.

---

### `lore_list`

Lists packages in a namespace or recent public packages.

**Input schema:**

```json
{
  "type": "object",
  "properties": {
    "namespace": {
      "type": "string",
      "description": "Namespace to list. Omit for recent public packages."
    },
    "kind": {
      "type": "string",
      "enum": ["Skill", "Rule", "Spec", "Template"]
    }
  }
}
```

---

### `lore_versions`

Lists all published versions of a package.

**Input schema:**

```json
{
  "type": "object",
  "required": ["package"],
  "properties": {
    "package": {
      "type": "string",
      "description": "Package in 'namespace/name' form.",
      "examples": ["m1cloud/azure-architect"]
    }
  }
}
```

---

## Resources

The server exposes packages as MCP resources under the `lore://` URI scheme.

**URI structure:**

```
lore://{namespace}/{name}:{version}
lore://{namespace}/{name}            ← resolves to latest
lore://{namespace}/{name}:latest     ← explicit latest
```

**URI template** (exposed in `resources/list` response):

```json
{
  "uriTemplate": "lore://{namespace}/{name}:{version}",
  "name": "Lore Package",
  "description": "Any package from the Lore Registry. Version may be 'latest' or an exact semver.",
  "mimeType": "text/markdown"
}
```

`resources/read` for any `lore://` URI returns the same content as `lore_get` with default parameters (server's `LORE_DEFAULT_LANG`, `include_deps=true`, active target rendering).

---

## Rendering Layer

Each target applies a minimal transform to the neutral markdown returned by the Registry. Rendering happens in the MCP Server, never in the Registry (see ADR-0008).

| Target | Transform applied |
|--------|------------------|
| `raw` | None. Registry markdown returned as-is. |
| `claude` | Adds a `<!-- lore:{kind}:{id} -->` HTML comment as the first line (non-rendered metadata annotation). |
| `cursor` | Prepends YAML frontmatter block for MDC compatibility (`description`, `alwaysApply: false`). |
| `copilot` | None currently (plain markdown is suitable for Copilot context injection). |
| `codex` | None currently (plain markdown). |
| `gemini` | None currently (plain markdown). |

The differences are minimal because the content is consumed inline as context, not written to a file. Platform-specific file installation (SKILL.md, MDC rules) is the CLI's responsibility; MCP handles ephemeral per-session injection.

**Cursor renderer example output:**

```
---
description: Azure Architect skill from Lore Registry (m1cloud/azure-architect:1.0.0)
alwaysApply: false
---

# Azure Architect

You are an expert Azure solutions architect...
```

---

## In-Memory Cache

The MCP Server maintains a TTL cache for fetched content to avoid redundant Registry API calls within a session.

- Cache key: `{id}:{lang}:{include_deps}`
- TTL: `LORE_CACHE_TTL` seconds (default 300)
- Eviction: LRU with a max of 100 entries
- The cache is process-local and does not persist across restarts

---

## Binary Distribution

The MCP Server is distributed as a single self-contained binary named `lore-mcp`, available for the same platforms as the CLI (`linux/amd64`, `linux/arm64`, `darwin/amd64`, `darwin/arm64`, `windows/amd64`).

It can also be run as a container for the hosted `mcp.lore.io` deployment.
