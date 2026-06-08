# Storage Architecture

The Registry stores namespaces and package artifacts.

Example:

```text
m1cloud/
├── namespace.yaml
├── README.md
└── packages/
    ├── azure-architect-1.0.0.lore
    ├── terraform-reviewer-1.1.0.lore
    └── security-baseline-2.4.1.lore
```

A .lore package is a compressed archive containing:

```text
manifest.yaml
README.md
LICENSE

content/
├── en-US.md
├── pt-BR.md

assets/

examples/
```

The Registry storage backend may be:

* Local Filesystem
* S3
* Azure Blob Storage
* MinIO
