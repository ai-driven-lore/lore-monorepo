# lore-monorepo
Draft initial

Knowledge as Package

"Lore" é universalmente compreendido como "corpo de conhecimento acumulado".

# Lore

> Universal registry for AI Skills, Rules and Specs.

Lore is an open-source platform for packaging, versioning, validating and distributing AI knowledge across development teams and organizations.

Inspired by Docker Hub, Helm Registry and Terraform Registry, Lore provides a standardized way to publish and consume reusable AI assets such as Skills, Rules, Specs, Templates and Agent Behaviors.

Instead of storing prompt files, instructions and specifications scattered across repositories, organizations can centralize their AI knowledge into a governed registry that can be consumed from any AI platform.

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

## Core Concepts

### Skill

Reusable capability that teaches an AI how to perform a specific task.

Examples:

* Azure Architect
* Kubernetes SRE
* Terraform Reviewer
* Security Auditor
* Solution Designer

### Rule

Behavioral constraints and policies.

Examples:

* Naming conventions
* Coding standards
* Security requirements
* Architectural constraints

### Spec

Structured project requirements.

Examples:

* SDD Specifications
* Architecture Documents
* Product Requirements
* Technical Standards

### Template

Reusable project scaffolding.

Examples:

* ADR Templates
* RFC Templates
* SDD Templates
* Design Documents

## Registry

Every asset is published into a Lore Registry.

Example:

lore.m1cloud.io/platform/azure-architect:v1.0.0

Similar to:

docker.io/library/nginx:latest

## CLI

Search available assets:

```bash
lore search azure
```

Inspect a package:

```bash
lore show platform/azure-architect
```

Install into a project:

```bash
lore pull platform/azure-architect
```

Validate local assets:

```bash
lore validate
```

Publish a new asset:

```bash
lore publish
```

## Project Structure

```text
.lore/
├── skills/
├── rules/
├── specs/
└── templates/
```

## Validation Engine

Lore validates every published asset.

Examples:

* Metadata validation
* Schema validation
* Dependency validation
* Prompt Injection detection
* Content quality validation
* Required translations validation

Invalid assets cannot be published.

## Internationalization

Lore supports multilingual assets.

Example:

```text
azure-architect/
├── en-US.md
├── pt-BR.md
└── es-ES.md
```

The CLI automatically installs the appropriate language based on user configuration.

## Dependencies

Assets may depend on other assets.

Example:

```yaml
dependencies:
  - platform/terraform-reviewer
  - platform/security-baseline
```

Install all dependencies:

```bash
lore install
```

## Future: Intelligent Discovery

Users should not need to know which Skills exist.

Future versions of Lore may analyze project context and recommend assets automatically.

Example:

```bash
lore recommend
```

Output:

```text
Recommended Skills:

✓ Azure Architect
✓ Terraform Reviewer
✓ Security Baseline
✓ Kubernetes SRE
```

Or:

```bash
lore solve "Create a production AKS cluster"
```

And Lore automatically discovers the most relevant Skills.

## MCP Integration

Lore can expose registry content through MCP servers, allowing AI agents to discover and consume organizational knowledge dynamically.

## Design Principles

* Vendor Agnostic
* Open Source
* Git Friendly
* Human Readable
* AI Native
* Versioned
* Governed
* Extensible

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

* AI Recommendations
* MCP Integration
* Asset Marketplace
* Reputation System

### Phase 4

* OCI Compatible Registry
* Enterprise Governance
* Federation

## License

Apache-2.0
