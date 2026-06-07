# Lore

> Registro universal para Skills, Rules e Specs de Inteligência Artificial.

Lore é uma plataforma open source para empacotar, versionar, validar e distribuir conhecimento utilizado por agentes de IA, equipes de desenvolvimento e organizações.

Inspirado em projetos como Docker Hub, Helm Registry e Terraform Registry, o Lore fornece uma maneira padronizada de publicar e consumir ativos reutilizáveis como Skills, Rules, Specs, Templates e comportamentos de agentes.

Em vez de armazenar instruções, prompts e especificações espalhados por diversos repositórios, as organizações podem centralizar seu conhecimento em um registro governado e consumi-lo a partir de qualquer plataforma de IA.

---

# Por que o Lore existe?

Hoje as equipes criam constantemente:

* Skills para agentes de IA
* Regras de desenvolvimento
* Padrões de arquitetura
* Especificações SDD
* Templates de documentação
* Prompts e instruções reutilizáveis

Na maioria dos casos, esses artefatos ficam:

* Espalhados em arquivos Markdown
* Duplicados entre projetos
* Difíceis de descobrir
* Difíceis de versionar
* Sem governança
* Dependentes de uma ferramenta específica

O Lore nasce para resolver esse problema.

---

# Visão

Tornar o conhecimento utilizado por IA portátil.

Uma Skill publicada uma única vez deve poder ser utilizada por:

* Claude Code
* OpenAI Codex
* GitHub Copilot
* Cursor
* Windsurf
* Continue
* MCP Clients
* Agentes personalizados

sem precisar ser reescrita para cada plataforma.

---

# Conceitos Fundamentais

## Skill

Uma capacidade reutilizável que ensina uma IA a executar uma tarefa específica.

Exemplos:

* Azure Architect
* Kubernetes SRE
* Terraform Reviewer
* Security Auditor
* Solution Designer

---

## Rule

Restrições comportamentais e diretrizes.

Exemplos:

* Convenções de nomenclatura
* Padrões de código
* Requisitos de segurança
* Restrições arquiteturais

---

## Spec

Especificações estruturadas que descrevem requisitos e processos.

Exemplos:

* SDD (Spec Driven Development)
* Arquiteturas de referência
* Requisitos de produto
* Padrões técnicos

---

## Template

Estruturas reutilizáveis para acelerar projetos.

Exemplos:

* ADR
* RFC
* Design Documents
* Templates de SDD

---

# Registry

Todos os ativos são publicados em um Registry Lore.

Exemplo:

```text
lore.m1cloud.io/platform/azure-architect:v1.0.0
```

De forma semelhante ao Docker:

```text
docker.io/library/nginx:latest
```

---

# CLI

Buscar ativos disponíveis:

```bash
lore search azure
```

Visualizar detalhes:

```bash
lore show platform/azure-architect
```

Instalar em um projeto:

```bash
lore pull platform/azure-architect
```

Validar ativos locais:

```bash
lore validate
```

Publicar um novo ativo:

```bash
lore publish
```

---

# Estrutura de Projeto

```text
.lore/
├── skills/
├── rules/
├── specs/
└── templates/
```

---

# Engine de Validação

Todo conteúdo publicado no Lore passa por validações automáticas.

Exemplos:

* Validação de metadados
* Validação de esquema
* Validação de dependências
* Detecção de Prompt Injection
* Validação de qualidade
* Verificação de traduções obrigatórias

Conteúdos inválidos não podem ser publicados.

---

# Internacionalização (i18n)

O Lore suporta múltiplos idiomas de forma nativa.

Exemplo:

```text
azure-architect/
├── pt-BR.md
├── en-US.md
└── es-ES.md
```

O CLI instala automaticamente a versão adequada conforme a configuração do usuário.

---

# Dependências

Ativos podem depender de outros ativos.

Exemplo:

```yaml
dependencies:
  - platform/terraform-reviewer
  - platform/security-baseline
```

Instalar dependências:

```bash
lore install
```

---

# Descoberta Inteligente (Futuro)

O usuário não deveria precisar saber previamente quais Skills existem.

No futuro, o Lore poderá analisar o contexto do projeto e sugerir automaticamente os ativos mais adequados.

Exemplo:

```bash
lore recommend
```

Resultado:

```text
Skills recomendadas:

✓ Azure Architect
✓ Terraform Reviewer
✓ Security Baseline
✓ Kubernetes SRE
```

Ou ainda:

```bash
lore solve "Criar um cluster AKS em produção"
```

E o Lore identifica automaticamente quais Skills, Rules e Specs devem ser utilizadas.

---

# Integração com MCP

O Lore poderá expor conteúdo através de servidores MCP, permitindo que agentes de IA descubram e consumam conhecimento organizacional dinamicamente.

---

# Princípios do Projeto

* Agnóstico de fornecedor
* Open Source
* Compatível com Git
* Legível por humanos
* Nativo para IA
* Versionado
* Governado
* Extensível

---

# Roadmap

## Fase 1

* Registry API
* CLI
* Empacotamento de Skills
* Engine de Validação

## Fase 2

* Resolução de Dependências
* Autenticação
* Namespaces Organizacionais
* Registries Privados

## Fase 3

* Recomendações por IA
* Integração MCP
* Marketplace de Ativos
* Sistema de Reputação

## Fase 4

* Compatibilidade OCI
* Governança Corporativa
* Federação entre Registries

---

# Licença

Apache 2.0
