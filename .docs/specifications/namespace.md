# Namespace Specification

Namespaces represent publishers inside a Lore Registry.

Examples:

* m1cloud
* community
* microsoft
* hashicorp

Namespace structure:

```text
m1cloud/
├── namespace.yaml
├── README.md
└── packages/
```

The `namespace.yaml` defines ownership, member roles, visibility and metadata for the namespace. It is the source of truth for both identity and authorization within that namespace.

Example:

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
```

For the full role definitions, permission matrix and resolution algorithm see the [RBAC Specification](./rbac.md).
