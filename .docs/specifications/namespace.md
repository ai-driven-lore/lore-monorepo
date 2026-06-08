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

The namespace manifest defines ownership, maintainers, metadata and policies.

Example:

```yaml
apiVersion: lore.io/v1

kind: Namespace

metadata:
  name: m1cloud

spec:
  displayName: M1 Cloud
  description: Cloud, Architecture and AI packages

maintainers:
  - moises
```
