# ADR-0005 - OIDC Authentication

Status: Accepted

## Context

The Lore Registry API requires an authentication mechanism for write operations (publishing packages, managing namespaces).

The Registry has no relational database. Namespaces and package artifacts are stored directly in a filesystem or object storage backend (S3, Azure Blob, MinIO). Any authentication strategy that requires server-side session or token storage would contradict this constraint.

The CLI is a first-class client. Browser-based OAuth redirect flows are not suitable for terminal environments.

Two approaches were considered:

**Satellite auth server (Docker Registry model):** the Registry delegates auth entirely to an external service. Simple core, but requires additional infrastructure to be operational.

**Embedded OIDC (configurable via environment variables):** the Registry validates tokens directly using the IdP's public keys. No satellite service required. IdP is pluggable.

## Decision

The Registry API SHALL authenticate requests using OIDC (OpenID Connect) with stateless JWT validation.

- The Registry SHALL NOT store tokens, sessions, or user records.
- Token validation SHALL use the IdP's JWKS endpoint to verify JWT signatures locally.
- The IdP SHALL be configurable via environment variables, supporting any OIDC-compliant provider (Google, GitHub, Azure AD, Okta, Keycloak, etc.).
- `LORE_AUTH_ENABLED` SHALL be required explicitly at startup. The Registry SHALL refuse to start if it is absent or set to an unrecognized value.
- When `LORE_AUTH_ENABLED=false`, the Registry SHALL operate in Development Mode: all write gates are open, RBAC is skipped, and the running state MUST be surfaced via startup logs, the `/_lore/info` endpoint, and an `X-Lore-Auth-Mode` response header.
- `LORE_AUTH_OIDC_CLIENT_SECRET` is NOT a registry variable. The Registry validates tokens via JWKS public keys and never handles the client secret. This credential is configured in the CLI and Web UI only.
- The CLI SHALL use the OAuth 2.0 Device Authorization Grant (RFC 8628) to authenticate without a browser redirect.
- Refresh tokens SHALL be stored locally on the user's machine (`~/.lore/credentials`), never on the server.
- Read operations on public packages SHALL be anonymous (no token required).
- SAML support is out of scope. Organizations requiring SAML can use an OIDC-compatible proxy (e.g., Keycloak) in front of their identity provider.

## Consequences

Benefits:

* Fully stateless — consistent with the filesystem-as-source-of-truth model
* No vendor lock-in — any OIDC-compliant IdP works without code changes
* Single binary deployment remains viable — no auth sidecar required
* CLI authentication is frictionless via Device Grant flow
* Public package consumption requires no credentials

Trade-offs:

* Immediate token revocation is not possible — compromised access tokens remain valid until expiration (typically 1 hour). Token introspection can be added in a future phase if required.
* The Registry depends on the IdP being reachable at startup to fetch JWKS keys. JWKS responses should be cached with a reasonable TTL to tolerate short IdP outages.
