---
name: sync-architecture
description: Use AFTER any modification to model.edn or views.edn. Regenerates all derived artifacts (diagrams, DSL, PlantUML). Run this before committing architecture changes.
requires:
  tools: [overarch, structurizr-cli]
  skills: [overarch-modeling]
skip_when:
  - No changes made to model.edn or views.edn
  - Creating initial model (use overarch-modeling first)
  - Only need one specific format (use structurizr-diagrams directly)
---

# Sync Architecture

Regenerates all architecture artifacts from the canonical Overarch model.

## Prerequisites

```bash
command -v overarch >/dev/null || { echo "Install: brew install overarch/brew/overarch"; exit 1; }
command -v structurizr-cli >/dev/null || { echo "Install: brew install structurizr-cli"; exit 1; }
```

## Workflow

### 1. Verify Model

```bash
ls vault/architecture/model.edn || echo "Run overarch-modeling first"
```

### 2. Validate Model

```bash
overarch -m vault/architecture/ --check
```

### 3. Regenerate Formats (Parallel)

**For faster execution, dispatch parallel subagents:**

```
Dispatch 3 subagents in parallel:

Subagent 1 - PNG Diagrams:
  overarch -m vault/architecture/ -r vault/architecture/diagrams/ -f png

Subagent 2 - Structurizr DSL:
  overarch -m vault/architecture/ -o vault/architecture/workspace.dsl -f structurizr

Subagent 3 - PlantUML:
  overarch -m vault/architecture/ -r vault/architecture/plantuml/ -f plantuml
```

If not parallelizing, run sequentially:
```bash
overarch -m vault/architecture/ -r vault/architecture/diagrams/ -f png
overarch -m vault/architecture/ -o vault/architecture/workspace.dsl -f structurizr
overarch -m vault/architecture/ -r vault/architecture/plantuml/ -f plantuml
```

### 4. Structurizr Export (Optional)

```bash
structurizr-cli export -workspace vault/architecture/workspace.dsl -format png
```

### 5. Check Threagile

```bash
if [ -f vault/security/threagile.yaml ]; then
    echo "Review: vault/security/threagile.yaml may need updates"
fi
```

### 6. Report Changes

```bash
git diff --stat vault/architecture/
```

## Artifact Mapping

| Source | Generated |
|--------|-----------|
| `model.edn` | `diagrams/*.png` |
| `model.edn` | `workspace.dsl` |
| `views.edn` | `diagrams/*.png` |

## When to Sync

- After editing `model.edn` or `views.edn`
- Before committing architecture changes
- Before architecture review

## Success Criteria

- [ ] `overarch -m vault/architecture/ --check` passes
- [ ] All diagram formats regenerated (PNG, DSL, PlantUML)
- [ ] `git diff --stat vault/architecture/` shows expected changes

## Related Skills

- `overarch-modeling` - Edit the source model
- `structurizr-diagrams` - Additional diagram options
