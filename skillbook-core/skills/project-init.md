---
name: project-init
description: Use when user asks to initialize a project, set up documentation structure, or configure skillbook tooling for a new codebase.
requires:
  tools: []
  skills: [init-obsidian-vault]
---

# Project Initialization

Guide for initializing a new project with skillbook capabilities.

## Project Folder Setup

**If not already in a project directory**, ask the user for:
1. Project name (used for folder name)
2. Whether to initialize git

```bash
# Create and enter project folder
mkdir -p <project-name>
cd <project-name>

# Optional: Initialize git
git init
```

**If already in a project directory** (has `.git/`, `deps.edn`, `package.json`, etc.), skip folder creation and proceed with initialization.

## Recommended Skills by Project Type

### Clojure Project

1. **Documentation**: `init-obsidian-vault` → `arc42-docs`
2. **Quality**: `clj-kondo-linting`, `cljfmt-formatting`, `splint-linting`
3. **Schemas**: `malli-schemas`
4. **Tasks**: `beads-tasks`

### Any Software Project

1. **Documentation**: `init-obsidian-vault`
2. **Architecture**: `overarch-modeling` → `structurizr-diagrams`
3. **Decisions**: `adr-management`
4. **Git**: `git-lint-setup`, `milestoner-releases`

## Parallel Initialization with Subagents

**For faster setup, dispatch independent skills in parallel:**

```
After running init-obsidian-vault (required first), dispatch parallel subagents:

Subagent 1 - Quality Tools:
  Run clj-kondo-linting setup
  Run cljfmt-formatting setup
  Run splint-linting setup

Subagent 2 - Git Workflow:
  Run git-lint-setup
  Run milestoner-releases setup

Subagent 3 - Architecture:
  Run overarch-modeling to create initial model
```

**Wait for all subagents, then run dependent skills:**
- `arc42-docs` (needs vault)
- `structurizr-diagrams` (needs overarch model)

## Skill Dependencies

```
init-obsidian-vault  ← Run first (creates vault/)
    ↓
┌───┴───┐
│       │
│   arc42-docs           (populates vault/arc42/)
│   adr-management       (uses vault/decisions/)
│   iso25010-quality     (uses vault/requirements/)
│
overarch-modeling    ← Run early (creates models/)
    ↓
structurizr-diagrams (reads models/)
sync-architecture    (regenerates from models/)
threagile-analysis   (informed by models/)

Independent (can run in parallel):
- clj-kondo-linting
- cljfmt-formatting
- splint-linting
- git-lint-setup
- milestoner-releases
- beads-tasks
```

## Quick Start Workflow

### 1. Initialize Documentation (Required First)

```bash
# Run init-obsidian-vault skill
```

### 2. Parallel Setup (Dispatch Subagents)

Run these in parallel via subagents:
- Quality tools setup
- Git workflow setup
- Architecture model creation

### 3. Dependent Skills (After Subagents Complete)

- `arc42-docs` - populate architecture documentation
- `structurizr-diagrams` - generate diagrams from model

## Output

After initialization:
```
vault/               # Documentation
├── arc42/          # Architecture docs
├── decisions/      # ADRs
└── requirements/   # Quality requirements

models/             # Architecture model (if using Overarch)
.clj-kondo/         # Linter config (if Clojure)
.cljfmt.edn         # Formatter config (if Clojure)
```

## Success Criteria

- [ ] `vault/` directory created with structure
- [ ] Selected quality tools configured
- [ ] Git workflow hooks installed (if selected)
- [ ] Architecture model created (if selected)
