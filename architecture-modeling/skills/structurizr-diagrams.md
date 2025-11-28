---
name: structurizr-diagrams
description: Use when EXPORTING diagrams to PNG/SVG/PlantUML/Mermaid from existing Structurizr DSL, or when Structurizr-specific features are needed (styling, deployment views). Requires workspace.dsl to exist.
requires:
  tools: [structurizr-cli]
  skills: [overarch-modeling]
skip_when:
  - No workspace.dsl exists yet (use sync-architecture to generate it first)
  - Only modifying the architecture model (use overarch-modeling)
  - PNG diagrams are sufficient (overarch can generate those directly)
---

# Structurizr Diagrams

## Prerequisites

```bash
command -v structurizr-cli >/dev/null || { echo "Install: brew install structurizr-cli"; exit 1; }
```

## Workflow

### 1. Read Overarch Model

```bash
ls models/*.edn
```

Extract: systems, containers, components, relationships.

### 2. Generate DSL

`architecture/workspace.dsl`:
```dsl
workspace "System Name" "Description" {
    model {
        user = person "User" "End user"
        system = softwareSystem "System" "Main system" {
            webapp = container "Web App" "Frontend" "React"
            api = container "API" "Backend" "Clojure"
            db = container "Database" "Storage" "PostgreSQL"
        }

        user -> webapp "Uses"
        webapp -> api "Calls"
        api -> db "Reads/writes"
    }

    views {
        systemContext system "Context" {
            include *
            autoLayout
        }
        container system "Containers" {
            include *
            autoLayout
        }
    }
}
```

### 3. Export Diagrams

```bash
# PNG images
structurizr-cli export -workspace workspace.dsl -format png

# PlantUML
structurizr-cli export -workspace workspace.dsl -format plantuml

# Mermaid
structurizr-cli export -workspace workspace.dsl -format mermaid
```

### 4. Validate

```bash
structurizr-cli validate -workspace workspace.dsl
```

## Quick DSL Reference

| Element | Syntax |
|---------|--------|
| Person | `name = person "Name" "Desc"` |
| System | `name = softwareSystem "Name" "Desc"` |
| Container | `name = container "Name" "Desc" "Tech"` |
| Component | `name = component "Name" "Desc" "Tech"` |
| Relationship | `source -> target "Description"` |

## View Types

| View | Shows |
|------|-------|
| `systemContext` | System and external actors |
| `container` | Containers within system |
| `component` | Components within container |
| `deployment` | Infrastructure nodes |
| `dynamic` | Runtime interactions |

## Reference Documentation

- `references/language.md` - Complete DSL syntax
- `references/views.md` - All view types
- `references/styling.md` - Themes and styles

## Success Criteria

- [ ] `workspace.dsl` validates without errors
- [ ] Diagrams exported to target format
- [ ] `structurizr-cli validate -workspace workspace.dsl` passes

## Related Skills

- `overarch-modeling` - Source model
- `sync-architecture` - Keep docs in sync
