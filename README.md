# Skillbook

A Claude Code marketplace for Design-First, Assurance-Driven software development.

## What is Skillbook?

Skillbook provides Claude Code plugins supporting a rigorous workflow that emphasizes formal verification (TLA+ via Recife) and continuous security checks (Threagile) before production code. All documentation lives in an Obsidian vault with arc42 architecture documentation.

## Plugins

### skillbook-core

Meta-orchestration for the skillbook ecosystem.

| Skill | Description |
|-------|-------------|
| `project-init` | Initialize projects with selected skills, run init skills in dependency order |

### project-management

Task tracking optimized for AI agents.

| Skill | Description |
|-------|-------------|
| `beads-tasks` | Git-native task management with Beads - dependency tracking, ready work detection |

### architecture-modeling

Central architecture model and visualizations.

| Skill | Description |
|-------|-------------|
| `overarch-modeling` | Create/maintain architecture in Overarch EDN format |
| `structurizr-diagrams` | Generate C4 diagrams from Overarch model |
| `sync-architecture` | Regenerate all derived formats from Overarch EDN |

### formal-verification

Formal specification and model checking.

| Skill | Description |
|-------|-------------|
| `tla-concepts` | Educational: TLA+ concepts, temporal logic, state machines |
| `recife-modeling` | Create Recife specs, run TLC, interpret counterexamples |

### security-compliance

Threat modeling and policy enforcement.

| Skill | Description |
|-------|-------------|
| `threagile-analysis` | Run Threagile threat modeling, report risks |
| `policy-as-code` | Generate OPA/Rego policies from Threagile mitigations |
| `conjtest-testing` | Validate IaC against policies using Conjtest |

### requirements-documentation

Requirements, specifications, and documentation in Obsidian.

| Skill | Description |
|-------|-------------|
| `init-obsidian-vault` | Set up Obsidian vault documentation structure |
| `arc42-docs` | Populate 12 arc42 sections with cross-links |
| `adr-management` | Create and manage Architectural Decision Records |
| `iso25010-quality` | Define quality requirements using ISO/IEC 25010 |
| `scenari-specs` | Create Gherkin specs with Scenari |

### clojure-quality

Linting, formatting, and style for Clojure.

| Skill | Description |
|-------|-------------|
| `splint-linting` | Set up Splint for idiom linting |
| `clj-kondo-linting` | Set up clj-kondo for static analysis |
| `cljfmt-formatting` | Set up cljfmt for code formatting |
| `clojure-style` | Coding conventions from Stuart Sierra and Clojure Style Guide |

### clojure-libraries

Clojure library patterns and best practices.

| Skill | Description |
|-------|-------------|
| `malli-schemas` | Data validation and generative testing with Malli |
| `guardrails-contracts` | Runtime contracts with Guardrails and guardrails-analyzer |

### git-workflow

Git conventions and release management.

| Skill | Description |
|-------|-------------|
| `git-lint-setup` | Git commit conventions (agents use gitmoji, humans commit freely) |
| `milestoner-releases` | Changelog and release management with Milestoner |

### infrastructure-ops

Infrastructure as Code and observability.

| Skill | Description |
|-------|-------------|
| `terraform-iac` | Generate Terraform from architecture, validate with policies |
| `slo-sli-observability` | Define SLOs/SLIs, generate observability code |

### skillcraft

Skills for creating AI skills.

| Skill | Description |
|-------|-------------|
| `skill-seekers` | Create skills from docs, GitHub repos, or PDFs |

### container-mastery

Docker optimization.

| Skill | Description |
|-------|-------------|
| `optimizing-dockerfiles` | Create optimized Dockerfiles with layer caching |

## Key Conventions

### Documentation Structure

All documentation lives in an Obsidian vault:

```
vault/
├── architecture/     # Overarch model, C4 diagrams
├── requirements/     # Quality attributes, stakeholders
├── specifications/   # Formal specs, feature embeds
├── decisions/        # ADRs
├── security/         # Threat model, policies
└── arc42/            # 12 architecture sections
```

### Commit Conventions

**Humans:** Commit freely, no rules.

**Agents:** Use `🤖 <gitmoji> <message>` with strict rules:
- ✨ Features, 🐛 Bug fixes, 🏗️ Architecture, 📝 Docs, 🧭 ADRs, 🔬 Formal specs, 🥒 BDD specs

### Architecture Flow

```
Overarch EDN (source of truth)
       ├──▶ Structurizr DSL ──▶ C4 Diagrams
       └──▶ Threagile YAML ──▶ Threat Analysis
```

## Installation

```bash
claude plugins install https://github.com/Kauko/skillbook.git
```

## License

MIT License - see [LICENSE](LICENSE) for details.
