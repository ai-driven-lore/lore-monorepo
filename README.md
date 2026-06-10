# Lore

> Universal registry for AI Skills, Rules and Specs.

Lore is an open-source platform for packaging, versioning, validating and distributing AI knowledge across development teams and organizations.

Inspired by Docker Hub, Helm Registry and Terraform Registry, Lore provides a standardized way to publish and consume reusable AI assets such as Skills, Rules, Specs, Templates and Agent Behaviors.

Instead of storing prompt files, instructions and specifications scattered across repositories, organizations can centralize their AI knowledge into a governed registry that can be consumed from any AI platform.

---

## Lore is not a prompt repository.

Lore is a distribution platform for AI knowledge.

---

## Why?

Today, teams create:

* Agent Skills
* Development Rules
* Coding Standards
* Architecture Patterns
* SDD Specifications
* Prompt Templates

Most of these assets are:

* Stored as loose Markdown files
* Duplicated across repositories
* Difficult to discover
* Difficult to version
* Difficult to govern
* Vendor specific

Lore aims to solve this problem by creating a universal distribution layer for AI knowledge.

---

## Vision

Make AI knowledge portable.

A Skill published once should be consumable from:

* Claude Code
* OpenAI Codex
* GitHub Copilot
* Cursor
* Windsurf
* Continue
* Custom Agents
* MCP Clients

without modifications.

---

## Core Concepts

### Skill

Reusable capability that teaches an AI how to perform a specific task.

### Rule

Behavioral constraints and policies.

### Spec

Structured project requirements.

### Template

Reusable project scaffolding.

---

## Registry

Every asset is published into a Lore Registry.

Example:

```text
lore.io/platform/azure-architect:v1.0.0
```

Similar to:

```text
docker.io/library/nginx:latest
```

---

## CLI

Search available assets:

```bash
lore search azure
```

Install into a project:

```bash
lore pull platform/azure-architect
```

Publish a new asset:

```bash
lore publish
```

---

## Future Vision

Users should not need to know which Skills exist.

```bash
lore solve "knowledge to terraform compliances"
```

Lore automatically discovers, resolves and installs the most relevant Skills, Rules and Specs for the requested outcome.

---

## Repository Structure

```text
lore/
├── apps/
│   ├── cli/               # Lore CLI
│   ├── pkg/               # Packages Shareds
│   │   ├── manifest/
│   │   ├── validator/
│   │   ├── packaging/
│   │   └── registry/
│   ├── registry/          # Registry API
│   ├── sdk/
│   │   ├── go/
│   │   ├── python/
│   │   ├── typescript/
│   │   ├── dotnet/
│   │   └── java/
│   ├── site/              # Web Site
│   ├── web/               # Registry Web UI
├── .docs/                 # Documentation
└── .examples/
├── infrastructure/
│   ├── helm/
│   ├── terraform/
│   └── kustomize/
```

### apps/registry - Registry API 

Responsible for:

* Package publishing
* Package retrieval
* Validation workflows
* Dependency resolution
* Registry APIs

### apps/cli - CLI

Primary developer experience.

### apps/web

Provides:

* Package discovery
* Search
* Documentation
* Registry administration

### apps/pkg - Shared Packages

Shared libraries used by both Registry and CLI.

### infrastructure

Deployment assets and examples.

Supported methods:

* Helm
* Terraform
* Kustomize

Future:

* Kubernetes Operator
* GitOps Integrations

---

## Validation Engine

Lore validates every published asset.

Examples:

* Metadata validation
* Schema validation
* Dependency validation
* Prompt Injection detection
* Required translations validation

Invalid assets cannot be published.

---

## Internationalization

Lore supports multilingual assets.

```text
azure-architect/
├── en-US.md
├── pt-BR.md
└── es-ES.md
```

---

## Technology Stack

| Component      | Technology                 |
| -------------- | -------------------------- |
| Registry API   | Go                         |
| CLI            | Go                         |
| Frontend       | Next.js                    |
| Infrastructure | Helm, Terraform, Kustomize |
| Deployment     | Kubernetes                 |

### Why Go?

Lore aims to be a cloud-native project focused on portability, performance and operational simplicity.

Using Go provides:

* Single binary deployments
* Cross-platform CLI distribution
* Low resource consumption
* Strong Kubernetes ecosystem integration
* Alignment with CNCF projects and tooling

---

## Roadmap

### Phase 1

* Registry API
* CLI
* Skill Packaging
* Validation Engine

### Phase 2

* Dependency Resolution
* Authentication
* Organization Namespaces
* Private Registries

### Phase 3

* MCP Integration
* AI-assisted Discovery
* Asset Marketplace

### Phase 4

* OCI Compatibility
* Enterprise Governance
* Registry Federation

---

## License

Apache-2.0
