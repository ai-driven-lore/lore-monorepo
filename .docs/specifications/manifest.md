# Manifest Specification

Every Lore Package MUST contain a manifest.yaml file.

Example:

```yaml
apiVersion: lore.io/v1

kind: Skill

metadata:
  namespace: m1cloud
  name: azure-architect
  version: 1.0.0

spec:
  title: Azure Architect
  description: Azure architecture specialist

content:
  defaultLanguage: en-US

translations:
  - en-US
  - pt-BR

dependencies:
  - m1cloud/terraform-reviewer
  - m1cloud/rule-naming-conventions

license: Apache-2.0

tags:
  - azure
  - architecture
```

Supported kinds:

* Skill
* Rule
* Spec
* Template

Future kinds:

* Agent
* Workflow
* MCPServer
