> ⚠️ Project Status

Lore is currently in the specification phase.

The package format, API contracts and architecture may change until the first stable release.

# Architecture Overview

Lore is a cloud-native platform for packaging, versioning, validating and distributing AI knowledge.

Lore is not tied to a specific AI provider.

A Lore Package contains knowledge in a platform-agnostic format and can be installed into different AI runtimes through adapters provided by the Lore CLI.

Core components:

* Registry API
* CLI
* Web UI
* Package Specification
* Validation Engine
* Authentication (OIDC)

The Registry stores package metadata and package artifacts.

The CLI is responsible for publishing, downloading, validating and rendering packages for supported AI runtimes.

The Web UI provides package discovery, documentation and administration capabilities.

Authentication is handled via OIDC. The Registry validates JWT tokens issued by a configurable external Identity Provider using stateless JWKS signature verification. No sessions or tokens are stored server-side. See the [Authentication Specification](./../specifications/auth.md) and [ADR-0005](./../decisions/adr-0005-oidc-authentication.md).
