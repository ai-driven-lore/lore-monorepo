# Lore

> Registro universal para Skills, Rules e Specs de Inteligência Artificial.

Lore é uma plataforma open source para empacotar, versionar, validar e distribuir conhecimento utilizado por agentes de IA, equipes de desenvolvimento e organizações.

Inspirado em Docker Hub, Helm Registry e Terraform Registry, o Lore fornece uma maneira padronizada de publicar e consumir ativos reutilizáveis como Skills, Rules, Specs, Templates e comportamentos de agentes.

Em vez de armazenar instruções e especificações espalhadas em diversos repositórios, as organizações podem centralizar seu conhecimento em um registro governado e consumi-lo a partir de qualquer plataforma de IA.

---

## Lore não é um repositório de prompts.

Lore é uma plataforma de distribuição de conhecimento para IA.

---

## Por que?

Hoje as equipes criam constantemente:

* Skills para agentes
* Regras de desenvolvimento
* Padrões de código
* Padrões arquiteturais
* Especificações SDD
* Templates reutilizáveis

Grande parte desses ativos fica:

* Espalhada em arquivos Markdown
* Duplicada entre projetos
* Difícil de descobrir
* Difícil de versionar
* Sem governança
* Dependente de ferramentas específicas

O Lore nasce para resolver esse problema.

---

## Visão

Tornar o conhecimento utilizado por IA portátil.

Uma Skill publicada uma única vez deve poder ser utilizada por:

* Claude Code
* OpenAI Codex
* GitHub Copilot
* Cursor
* Windsurf
* Continue
* Agentes personalizados
* Clientes MCP

sem precisar ser adaptada para cada plataforma.

---

## Conceitos Fundamentais

### Skill

Capacidade reutilizável que ensina uma IA a executar uma tarefa específica.

### Rule

Diretrizes e restrições comportamentais.

### Spec

Especificações estruturadas de projeto.

### Template

Estruturas reutilizáveis para acelerar projetos.

---

## Registry

Todo ativo é publicado em um Registry Lore.

Exemplo:

```text
lore.io/platform/azure-architect:v1.0.0
```

Semelhante a:

```text
docker.io/library/nginx:latest
```

---

## CLI

Buscar ativos:

```bash
lore search azure
```

Instalar em um projeto:

```bash
lore pull platform/azure-architect
```

Publicar um ativo:

```bash
lore publish
```

---

## Visão de Futuro

O usuário não deveria precisar saber previamente quais Skills existem.

```bash
lore solve "knowledge to terraform compliances"
```

O Lore identifica automaticamente as Skills, Rules e Specs mais adequadas para atingir o objetivo solicitado.

---

## Estrutura do Repositório

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
│   ├── site/              # Web Site
│   ├── web/               # Registry Web UI
├── .docs/                 # Documentation
└── examples/
├── infrastructure/
│   ├── helm/
│   ├── terraform/
│   └── kustomize/
```

### apps/registry - Registry

Responsável por:

* Publicação de pacotes
* Download de pacotes
* Validação
* Resolução de dependências
* APIs do Registry

### apps/cli - CLI

Principal interface para desenvolvedores.

### apps/web

Responsável por:

* Descoberta de ativos
* Busca
* Documentação
* Administração

### apps/pkg - Shared Packages

Bibliotecas compartilhadas entre Registry e CLI.

### infrastructure

Recursos de implantação.

Métodos suportados:

* Helm
* Terraform
* Kustomize

Futuro:

* Kubernetes Operator
* Integrações GitOps

---

## Engine de Validação

Todo conteúdo publicado passa por validações automáticas.

Exemplos:

* Validação de metadados
* Validação de esquema
* Validação de dependências
* Detecção de Prompt Injection
* Verificação de traduções obrigatórias

---

## Internacionalização

O Lore suporta múltiplos idiomas.

```text
azure-architect/
├── pt-BR.md
├── en-US.md
└── es-ES.md
```

---

## Stack Tecnológica

| Componente     | Tecnologia                 |
| -------------- | -------------------------- |
| Registry API   | Go                         |
| CLI            | Go                         |
| Frontend       | Next.js                    |
| Infraestrutura | Helm, Terraform, Kustomize |
| Implantação    | Kubernetes                 |

---

## Roadmap

### Fase 1

* Registry API
* CLI
* Empacotamento de Skills
* Engine de Validação

### Fase 2

* Resolução de Dependências
* Autenticação
* Namespaces Organizacionais
* Registries Privados

### Fase 3

* Integração MCP
* Descoberta Assistida por IA
* Marketplace de Ativos

### Fase 4

* Compatibilidade OCI
* Governança Corporativa
* Federação de Registries

---

## Licença

Apache-2.0
