# ADR-0009: Introduce `/v1/content/` Extraction API in the Registry

**Status:** Accepted

**Date:** 2026-06-10

---

## Context

The Lore Registry stores packages as `.lore` archives in object storage. The existing download endpoint (`GET /v1/packages/{ns}/{name}/{version}/raw`) returns the full compressed archive — the right interface for the CLI (`lore pull`) which needs the complete package locally.

However, online consumers — the MCP Server, the Web UI, and future REST integrations — need to read individual fields from packages (manifest metadata, body content in a specific language) without downloading and decompressing a full archive for each request. Doing so on every request would be slow, wasteful, and put unnecessary load on storage.

Two approaches were considered:

**Option A — No new API. Consumers download and decompress the archive themselves.**

Every MCP Server `lore_get` call downloads the `.lore` archive, decompresses it in memory, reads the relevant files, and discards the archive. This works but has significant drawbacks: repeated bandwidth consumption, high latency for large packages, and logic duplication across consumers.

**Option B — Introduce a `/v1/content/` extraction API.**

The Registry exposes endpoints that serve pre-extracted content fields directly. The Registry handles decompression once (on first access) and caches the extracted content in storage for subsequent requests.

---

## Decision

Introduce a `/v1/content/` surface in the Registry API. See [content-api.md](../specifications/content-api.md) for the full endpoint specification.

---

## Consequences

**Benefits:**

- **Low-latency content access.** After the first request for a version, all subsequent content reads are served from extracted files in storage — no archive decompression in the hot path.
- **Single extraction point.** The Registry is the only component that decompresses `.lore` archives at runtime. The MCP Server, Web UI, and Copilot Extension all read from the same extracted content without each needing packaging library dependencies.
- **Correct caching semantics.** Package versions are immutable after publication. Extracted content is permanently valid while the version exists. The Registry can set `Cache-Control: immutable` on content responses, enabling aggressive CDN and proxy caching.
- **Clean separation of concerns.** The `/raw` endpoint remains the download path for the CLI. The `/content/` endpoints are the read path for online consumers. Each path is optimized for its use case.
- **Dependency resolution centralized.** When `include_deps=true` is requested, the Registry resolves the dependency graph and returns a flat ordered list. Consumers do not need to implement dependency resolution themselves.

**Trade-offs:**

- **Storage amplification.** For each published package version, the Registry stores both the `.lore` archive and the extracted sidecar files. The sidecar files are typically smaller than the archive (markdown text compresses well), but storage use increases. Mitigation: sidecar files are created lazily (on first access), so rarely-accessed packages incur no extra storage cost until they are requested.
- **Extraction latency on first access.** The first request for a package's content will be slower than subsequent requests while extraction is performed. This is a cold-start cost that most consumers will experience at most once per package version. Mitigation: the CLI's `lore publish` command can optionally pre-warm the cache immediately after publishing.
- **API surface grows.** The Registry adds new endpoints to maintain. Mitigation: the `/v1/content/` surface is simple and read-only; it does not require auth for public packages and adds no write-path complexity.
