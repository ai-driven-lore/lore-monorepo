# Authentication Specification

Lore Registry API authentication is stateless. The server holds no sessions, user records, or token storage. All identity information is derived from JWT tokens issued by an external OIDC-compliant Identity Provider (IdP).

See [ADR-0005](./../decisions/adr-0005-oidc-authentication.md) for the rationale behind this decision.

---

## Public vs. Protected Operations

| Operation | Authentication Required |
|---|---|
| `GET /packages/:namespace/:name/:version` | No — anonymous |
| `GET /namespaces/:namespace` | No — anonymous |
| `GET /search` | No — anonymous |
| `POST /packages` (publish) | Yes |
| `DELETE /packages/:namespace/:name/:version` | Yes |
| `POST /namespaces` | Yes |
| `PATCH /namespaces/:namespace` | Yes |

Public read access requires no credentials. This removes friction for package consumers and aligns with the behavior of Docker Hub, Helm Registry, and Terraform Registry.

---

## Token Validation

Every protected request MUST include a Bearer token in the Authorization header:

```
Authorization: Bearer <access_token>
```

The Registry validates the token by:

1. Fetching the IdP's JWKS public keys from the discovery endpoint (`/.well-known/openid-configuration`)
2. Verifying the JWT signature locally
3. Checking `iss`, `aud`, and `exp` claims

JWKS keys are cached by the Registry to avoid a network round-trip on every request and to tolerate short IdP unavailability.

No token data is persisted. The Registry is stateless between requests.

---

## Identity Provider Configuration

The IdP is configured entirely via environment variables:

```env
LORE_AUTH_ENABLED=true
LORE_AUTH_OIDC_ISSUER=https://accounts.google.com
LORE_AUTH_OIDC_CLIENT_ID=<client-id>
LORE_AUTH_OIDC_CLIENT_SECRET=<client-secret>
LORE_AUTH_OIDC_AUDIENCE=lore-registry
LORE_AUTH_ALLOW_ANONYMOUS_READ=true
```

Supported OIDC-compliant providers include: Google, GitHub, Azure AD, Okta, Keycloak, and any provider exposing a standard `/.well-known/openid-configuration` endpoint.

Organizations using SAML-only providers can introduce an OIDC proxy such as Keycloak without changes to the Registry.

---

## CLI Authentication Flow

The CLI uses the **OAuth 2.0 Device Authorization Grant** (RFC 8628), which does not require a browser redirect and is suitable for terminal environments.

```bash
lore login
```

```
→ Open https://accounts.google.com/device and enter the code: XKCD-9821
→ Waiting for authorization...
✓ Authenticated as moises@mmgil.com.br
```

After authorization the CLI receives:

- `access_token` — short-lived JWT (~1h), sent in API requests
- `refresh_token` — long-lived, stored locally in `~/.lore/credentials`

The `refresh_token` is stored **only on the user's machine**. The Registry never sees or stores it.

When the `access_token` expires, the CLI silently exchanges the `refresh_token` for a new one directly with the IdP. The user is not prompted again.

---

## Web UI Authentication Flow

The Web UI uses the standard **OIDC Authorization Code Flow** with browser redirect. The UI exchanges the authorization code for tokens and uses the `access_token` for API calls.

---

## Authorization Model

Authentication establishes identity. Authorization — determining what an authenticated identity is allowed to do — is governed by the RBAC model defined in the [RBAC Specification](./rbac.md).

In short: the Registry extracts the identity from the JWT, reads the target `namespace.yaml` from storage, and checks whether the identity has a role that permits the requested operation. No separate authorization database is required.

Global administrators are declared in `registry.yaml` at the storage root and bypass namespace-level role checks.

---

## Token Revocation

Immediate revocation is not supported in this model. A compromised `access_token` remains valid until its `exp` claim is reached.

Mitigation: keep access token TTL short (≤1h, delegated to the IdP configuration). The refresh token can be revoked directly at the IdP.

Token introspection (real-time revocation check via the IdP) is a candidate for a future phase if the threat model requires it.
