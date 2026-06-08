# RBAC Specification

Authorization in the Lore Registry is role-based and embedded in the filesystem. Permissions are resolved by reading `namespace.yaml` or `registry.yaml` from storage on each protected request. No separate authorization database or policy file exists.

See [ADR-0006](./../decisions/adr-0006-rbac-namespace.md) for the rationale behind this decision.

---

## Roles

### Namespace Roles

| Role | Scope |
|---|---|
| `owner` | Full control over the namespace: publish, delete packages, manage members, update namespace metadata, delete the namespace |
| `publisher` | Publish and update packages within the namespace |
| `viewer` | Read access to packages in private namespaces (has no effect on public namespaces, where anonymous read is already allowed) |

### Registry Role

| Role | Scope |
|---|---|
| `admin` | Registry-wide: create and delete any namespace, manage any namespace's members, access private namespaces |

---

## Permission Matrix

| Operation | Anonymous | Viewer | Publisher | Owner | Admin |
|---|:---:|:---:|:---:|:---:|:---:|
| Read package (public namespace) | ✓ | ✓ | ✓ | ✓ | ✓ |
| Read package (private namespace) | ✗ | ✓ | ✓ | ✓ | ✓ |
| Search packages | ✓ | ✓ | ✓ | ✓ | ✓ |
| Publish package | ✗ | ✗ | ✓ | ✓ | ✓ |
| Delete package | ✗ | ✗ | ✗ | ✓ | ✓ |
| Read namespace metadata | ✓ * | ✓ | ✓ | ✓ | ✓ |
| Update namespace metadata | ✗ | ✗ | ✗ | ✓ | ✓ |
| Manage namespace members | ✗ | ✗ | ✗ | ✓ | ✓ |
| Delete namespace | ✗ | ✗ | ✗ | ✓ | ✓ |
| Create namespace | ✗ ** | ✗ | ✗ | ✗ | ✓ ** |

\* Anonymous read of namespace metadata is allowed only for public namespaces.

\*\* Namespace creation by authenticated non-admin users can be enabled via `registry.yaml` (`openNamespaceCreation: true`), which is the default for public registry deployments.

---

## namespace.yaml — RBAC Fields

Every `namespace.yaml` MUST declare its `visibility` and a `members` list.

```yaml
apiVersion: lore.io/v1
kind: Namespace

metadata:
  name: m1cloud

spec:
  displayName: M1 Cloud
  description: Cloud, Architecture and AI packages
  visibility: public   # public | private

members:
  - identity: moises@mmgil.com.br
    role: owner
  - identity: rodrigo@example.com
    role: publisher
  - identity: partner@example.com
    role: viewer
```

**`visibility`**

- `public` — packages are readable by anyone without authentication. `viewer` roles have no effect.
- `private` — packages are readable only by authenticated members and global admins.

**`members[].identity`**

The value matched against the `email` claim in the JWT issued by the IdP. The claim used for matching is configurable via `LORE_AUTH_IDENTITY_CLAIM` (default: `email`).

**`members[].role`**

One of `owner`, `publisher`, or `viewer`.

A namespace MUST have at least one `owner` at all times. The Registry SHALL reject member updates that would leave a namespace without an owner.

---

## registry.yaml — Global Administrators

A `registry.yaml` file at the storage root defines registry-level configuration, including global administrators.

```yaml
apiVersion: lore.io/v1
kind: RegistryConfig

settings:
  openNamespaceCreation: true   # any authenticated user can create a namespace

admins:
  - moises@mmgil.com.br
```

**`settings.openNamespaceCreation`**

When `true` (default for public deployments), any authenticated user can create a namespace. When `false`, only global admins can create namespaces. Useful for private or enterprise registry deployments.

**`admins`**

List of identities with registry-wide admin privileges. These users bypass namespace-level role checks and can operate on any namespace.

---

## Permission Resolution

On each protected request the Registry resolves permissions as follows:

```
1. Extract identity from JWT claim (default: email)
2. Check registry.yaml admins → if match, allow
3. Identify target namespace from request
4. Read namespace.yaml from storage
5. Find member entry matching the identity
6. Check if the member's role permits the requested operation
7. Allow or reject
```

No in-memory permission state is held between requests. The namespace.yaml read in step 4 may be served from a short-lived cache (recommended TTL: 30s) to avoid redundant storage I/O on high-frequency operations.

---

## Identity Claim Configuration

By default the Registry matches member identities against the `email` JWT claim. This can be changed if the IdP uses a different claim (e.g., `preferred_username`, `sub`):

```env
LORE_AUTH_IDENTITY_CLAIM=email
```

The value in `members[].identity` in `namespace.yaml` must match the exact value of the configured claim in the token.
