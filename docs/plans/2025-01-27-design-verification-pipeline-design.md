# Design & Verification Pipeline Skills

## Overview

A collection of Claude Code skills supporting a Design-First, Assurance-Driven Workflow for building complex software. Focuses on formal verification (TLA+ via Recife) and continuous security checks (Threagile) before production code.

## Plugins and Skills

### skillbook-core

Meta-orchestration for the skillbook ecosystem.

| Skill | Purpose |
|-------|---------|
| `project-init` | Discovers installed skills, lets user select which to initialize for a repo, runs init skills in dependency order, generates `.skillbook/manifest.edn` |

### project-management

Task tracking optimized for AI agents.

| Skill | Purpose |
|-------|---------|
| `beads-tasks` | Git-native task management with Beads; dependency tracking, `bd ready` for next tasks |

### architecture-modeling

Central architecture model and visualizations.

| Skill | Purpose |
|-------|---------|
| `overarch-modeling` | Create/maintain architecture in Overarch EDN format at `vault/architecture/model.edn` |
| `structurizr-diagrams` | Generate C4 diagrams from Overarch, output to `vault/architecture/diagrams/` |
| `sync-architecture` | Regenerate all derived formats (Structurizr DSL, Threagile YAML) from Overarch EDN |

### formal-verification

Formal specification and model checking.

| Skill | Purpose |
|-------|---------|
| `tla-concepts` | Educational: TLA+ concepts, temporal logic, state machines, invariants |
| `recife-modeling` | Create Recife specs, translate Gherkin to predicates, run TLC, interpret counterexamples |

### security-compliance

Threat modeling and policy enforcement.

| Skill | Purpose |
|-------|---------|
| `threagile-analysis` | Run Threagile threat modeling, report risks, output to `vault/security/` |
| `policy-as-code` | Generate OPA/Rego policies from Threagile mitigations |
| `conjtest-testing` | Validate IaC against policies using Conjtest (Clojure Conftest wrapper) |

### requirements-documentation

Requirements, specifications, and documentation in Obsidian.

| Skill | Purpose |
|-------|---------|
| `init-obsidian-vault` | Set up `vault/` structure with folders for architecture, requirements, decisions, security, arc42 |
| `arc42-docs` | Populate 12 arc42 sections, cross-link to other artifacts |
| `adr-management` | Create/manage Architectural Decision Records in `vault/decisions/` |
| `iso25010-quality` | Define quality requirements using ISO/IEC 25010 taxonomy |
| `scenari-specs` | Create Gherkin specs with Scenari in `test/features/`, embed in vault |

### clojure-quality

Linting, formatting, and style for Clojure code.

| Skill | Purpose |
|-------|---------|
| `splint-linting` | Set up Splint, CLI scripts, optional git hook |
| `clj-kondo-linting` | Set up clj-kondo, auto-import library configs, CLI scripts, optional git hook |
| `cljfmt-formatting` | Set up cljfmt, CLI scripts, optional git hook |
| `clojure-style` | Coding conventions: Stuart Sierra's rules, Clojure Style Guide, custom rules (e.g., `::not-found` fallback) |

### clojure-libraries

Clojure library patterns and best practices.

| Skill | Purpose |
|-------|---------|
| `malli-schemas` | Data validation, schemas, generative testing with Malli |
| `guardrails-contracts` | Runtime contracts with Guardrails + static analysis with guardrails-analyzer |

### git-workflow

Git conventions and release management.

| Skill | Purpose |
|-------|---------|
| `git-lint-setup` | Set up git-lint for agent commits only; humans commit freely |
| `milestoner-releases` | Changelog and release management with Milestoner |

### infrastructure-ops

Infrastructure as Code and observability.

| Skill | Purpose |
|-------|---------|
| `terraform-iac` | Generate Terraform from architecture, validate with policies |
| `slo-sli-observability` | Define SLOs/SLIs, generate observability code snippets |

## Key Conventions

### Documentation Structure

All documentation lives in Obsidian vault:

```
vault/
├── README.md
├── architecture/
│   ├── model.edn (Overarch - single source of truth)
│   ├── diagrams/
│   └── diagrams.md
├── requirements/
│   ├── quality-attributes.md
│   └── stakeholders.md
├── specifications/
│   ├── features.md (embeds test/features/*.feature)
│   └── formal/
├── decisions/
│   └── NNNN-title.md (ADRs)
├── security/
│   ├── threat-model.md
│   └── policies.md
└── arc42/
    └── (12 section files)
```

### Feature Files

- Source of truth: `test/features/*.feature`
- Embedded in vault: `![[../test/features/auth.feature]]`

### Architecture Flow

```
Overarch EDN (source of truth)
       │
       ├──▶ Structurizr DSL ──▶ C4 Diagrams
       │
       └──▶ Threagile YAML ──▶ Threat Analysis
```

Run `sync-architecture` after editing Overarch to regenerate derived formats.

### Commit Conventions

Two-tier system:

**Humans:** No rules. Commit freely.

**Agents:** Must use `🤖 <gitmoji> <message>` format with strict rules:

| Category | Emoji | Use |
|----------|-------|-----|
| Feature | ✨ | New features |
| Bug fix | 🐛 | Bug fixes |
| Hotfix | 🚑️ | Critical fixes |
| Refactor | ♻️ | Refactoring |
| Tests | ✅ | Test changes |
| Architecture | 🏗️ | Architectural changes |
| Security | 🔒️ | Security fixes |
| Docs | 📝 | Documentation |
| ADR | 🧭 | Decision records |
| Formal spec | 🔬 | TLA+/Recife specs |
| BDD spec | 🥒 | Gherkin scenarios |
| Types/Schemas | 🏷️ | Malli schemas |
| Validation | 🦺 | Guardrails contracts |
| Infrastructure | 🧱 | IaC changes |
| Config | 🔧 | Configuration |
| Release | 🔖 | Version tags |

Agent rules:
- Single concern per commit
- No mixing code + docs
- Scope required for code: `🤖 ✨ (orders) Add bulk processing`

### Tool Availability

All skills:
1. Check if tool is installed
2. Guide installation if missing
3. Proceed with workflow

### Skill Depth

- Starter templates for structural artifacts
- Workflow guidance as primary focus
- Educational tone where appropriate (especially formal-verification)

## Plugin Dependencies

```
skillbook-core
    └── project-init (orchestrates all other init skills)

requirements-documentation
    └── init-obsidian-vault (required before arc42-docs, adr-management)

architecture-modeling
    ├── overarch-modeling (required before sync-architecture)
    └── sync-architecture (required before structurizr-diagrams, threagile-analysis)

security-compliance
    ├── threagile-analysis (required before policy-as-code)
    └── policy-as-code (required before conjtest-testing)
```
