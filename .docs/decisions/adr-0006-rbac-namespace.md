# ADR-0006 - RBAC Embedded in namespace.yaml

Status: Accepted

## Context

The Lore Registry requires an authorization model that controls who can publish, update, delete and manage packages and namespaces.

Two approaches were evaluated:

**External policy file (Casbin/policy.csv):** a centralized file, similar to the approach used by ArgoCD, that expresses all permissions for all namespaces in a single artifact. Casbin supports rich policy expressions (`p, role, resource, action, effect`).

**RBAC embedded in namespace.yaml:** each namespace declares its own members and their roles directly inside its `namespace.yaml`, which already exists in the storage layer as the namespace's source of truth.

The external policy file approach was discarded for three reasons:

1. It centralizes permissions for all namespaces into a single file, creating a management bottleneck and coupling unrelated namespaces together.
2. It is difficult to manage through the Web UI — exposing Casbin syntax to end users is a poor experience.
3. It introduces a new artifact type with no clear ownership in the storage model.

## Decision

RBAC SHALL be expressed directly inside each namespace's `namespace.yaml` as a `members` list with role assignments.

A separate `registry.yaml` at the storage root SHALL define global administrators for registry-wide operations.

The Registry SHALL resolve permissions by reading the relevant `namespace.yaml` (or `registry.yaml`) from storage on each protected request. No in-memory permission cache with long TTL, no separate policy store.

## Consequences

Benefits:

* Consistent with the filesystem-as-source-of-truth model — no new storage concept introduced
* Each namespace is self-contained and independently manageable
* Web UI can render a member management interface without exposing YAML syntax
* Auditable via version control if namespace files are tracked in git
* Stateless resolution — the server reads a file, checks membership, responds

Trade-offs:

* Permission changes take effect on the next request after the file is updated (no instant propagation, acceptable given short JWKS cache TTL)
* No cross-namespace role expressions (e.g., "publisher in all namespaces under org X") — if this becomes necessary in a future phase, a higher-level `organization.yaml` can be introduced above the namespace level
