# Content API Specification

The Registry exposes a `/v1/content/` surface for online consumption of package content. Unlike the `/v1/packages/{ns}/{name}/{version}/raw` endpoint (which downloads the full `.lore` archive for CLI use), the Content API extracts and serves individual fields from inside archives on demand.

## Design Principles

- The Registry returns **neutral markdown only** — no platform-specific rendering. Rendering is the responsibility of consumers (see [mcp-server.md](./mcp-server.md)).
- Package versions are **immutable** after publication. Content is safe to cache indefinitely per version.
- First access triggers extraction; subsequent requests are served from the content cache.
- Auth follows the same rules as the rest of the Registry API (see [auth.md](./auth.md)): public packages require no auth; private packages require a valid OIDC JWT.

---

## Endpoints

### `GET /v1/content/{namespace}/{name}/{version}`

Returns the full content metadata for a package: manifest fields, available translations, and the body in the requested language.

**Path parameters:**
- `namespace` — e.g., `m1cloud`
- `name` — e.g., `azure-architect`
- `version` — exact semver (`1.0.0`), `latest`, or a semver range (`^1.0`, `~1.0.0`, `1.0.x`). Non-exact versions resolve to the highest matching version and respond with `303 See Other` pointing to the canonical URL.

**Query parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `lang` | string (BCP 47) | `defaultLanguage` from manifest | Translation to return (`en-US`, `pt-BR`, etc.) |
| `include_deps` | boolean | `false` | When `true`, resolved dependencies are inlined in the response |

**Response `200 OK`:**

```json
{
  "manifest": {
    "apiVersion": "lore.io/v1",
    "kind": "Skill",
    "metadata": {
      "namespace": "m1cloud",
      "name": "azure-architect",
      "version": "1.0.0"
    },
    "spec": {
      "title": "Azure Architect",
      "description": "Azure architecture specialist"
    },
    "content": {
      "defaultLanguage": "en-US",
      "translations": ["en-US", "pt-BR"]
    },
    "dependencies": ["m1cloud/terraform-reviewer:1.1.0"],
    "license": "Apache-2.0",
    "tags": ["azure", "architecture"]
  },
  "body": "# Azure Architect\n\nYou are an expert...",
  "language": "en-US",
  "dependencies_resolved": [
    {
      "id": "m1cloud/terraform-reviewer:1.1.0",
      "title": "Terraform Reviewer",
      "body": "# Terraform Reviewer\n\nYou are an expert..."
    }
  ]
}
```

When `include_deps=false`, `dependencies_resolved` is an empty array.

**Error responses:**

| Status | Condition |
|--------|-----------|
| `404 Not Found` | Package or version does not exist |
| `404 Not Found` | Requested `lang` is not available; response body lists available translations |
| `401 Unauthorized` | Package is in a private namespace and no valid token was provided |
| `403 Forbidden` | Token provided but caller lacks read access to the namespace |
| `303 See Other` | Version was a range/`latest`; `Location` header contains the resolved canonical URL |

---

### `GET /v1/content/{namespace}/{name}/{version}/body`

Returns only the raw markdown body in the requested language. No JSON wrapper.

**Query parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `lang` | string (BCP 47) | `defaultLanguage` from manifest | Translation to return |

**Response `200 OK`:**
- `Content-Type: text/markdown; charset=utf-8`
- Body: the raw markdown content of the package

This is the fastest path for consumers that only need the body text (e.g., the MCP Server's internal fetcher). No JSON serialization overhead.

---

### `GET /v1/packages/{namespace}/{name}/versions`

Lists all published versions of a package, sorted by semver descending.

**Response `200 OK`:**

```json
{
  "namespace": "m1cloud",
  "name": "azure-architect",
  "versions": [
    { "version": "1.0.0", "published_at": "2026-01-15T10:00:00Z", "latest": true },
    { "version": "0.9.0", "published_at": "2025-11-01T08:00:00Z", "latest": false }
  ]
}
```

---

### `GET /v1/search`

Full-text search across all public packages (and private packages the caller has access to).

**Query parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `q` | string | — | Search query. Matches title, description, tags. |
| `kind` | string | — | Filter: `Skill`, `Rule`, `Spec`, `Template` |
| `namespace` | string | — | Restrict to a specific namespace |
| `tag` | string (repeatable) | — | Filter by tag. Multiple `?tag=` values are AND-combined. |
| `lang` | string (BCP 47) | — | Return only packages that have this translation |
| `page` | integer | `0` | 0-indexed page number |
| `limit` | integer | `20` | Results per page (max: `100`) |

**Response `200 OK`:**

```json
{
  "total": 42,
  "page": 0,
  "limit": 20,
  "results": [
    {
      "id": "m1cloud/azure-architect:1.0.0",
      "namespace": "m1cloud",
      "name": "azure-architect",
      "version": "1.0.0",
      "kind": "Skill",
      "title": "Azure Architect",
      "description": "Azure architecture specialist",
      "tags": ["azure", "architecture"],
      "translations": ["en-US", "pt-BR"],
      "published_at": "2026-01-15T10:00:00Z"
    }
  ]
}
```

No auth required for public packages. Private packages appear only if the caller has a valid token with at least `viewer` role in that namespace.

---

## Content Cache Strategy

The Registry stores `.lore` archives in object storage. Decompressing an archive on every `/v1/content/` request is not acceptable.

**Extraction on first access:** When a content request arrives for a version with no cached extraction, the Registry:
1. Fetches the `.lore` archive from storage
2. Extracts `manifest.yaml` and all `content/*.md` files
3. Writes extracted content to a sidecar path in storage: `{namespace}/cache/{name}-{version}/manifest.json` and `{namespace}/cache/{name}-{version}/{lang}.md`
4. Serves the response from the newly extracted content

**Cache invalidation:** Package versions are immutable after publication. Extracted content is never stale while the package version exists. When a version is deleted, the sidecar path is deleted alongside the archive.

**Dependency resolution:** When `include_deps=true`, the Registry resolves the dependency graph server-side, returning a flat ordered list (dependencies before dependents). Cycle detection is enforced; a circular dependency chain returns `422 Unprocessable Entity`.
