---
name: overarch-modeling
description: Use when CREATING or MODIFYING the canonical architecture model (model.edn, views.edn). This is the source of truth - use BEFORE structurizr-diagrams or sync-architecture.
requires:
  tools: [overarch]
  skills: []
skip_when:
  - Only generating diagrams from existing model (use sync-architecture)
  - Only exporting to different formats (use structurizr-diagrams)
  - Model already exists and no changes needed
  - Working on threat modeling only (use threagile-analysis)
---

# Overarch Modeling

## Prerequisites

```bash
command -v overarch >/dev/null || { echo "Install: brew install overarch/brew/overarch"; exit 1; }
```

## Workflow

### 1. Create/Update Model

`models/model.edn`:
```clojure
#{;; System context
  {:el :person
   :id :user
   :name "User"
   :desc "End user of the system"}

  {:el :system
   :id :main-system
   :name "Main System"
   :desc "The system being modeled"
   :ct #{:webapp :api :database}}

  ;; Containers
  {:el :container
   :id :webapp
   :name "Web Application"
   :tech "React"
   :desc "User interface"}

  {:el :container
   :id :api
   :name "API Server"
   :tech "Clojure"
   :desc "Business logic"}

  {:el :container
   :id :database
   :name "Database"
   :tech "PostgreSQL"
   :subtype :database}

  ;; Relationships
  {:el :request
   :id :user-webapp
   :from :user
   :to :webapp
   :name "Uses"}

  {:el :request
   :id :webapp-api
   :from :webapp
   :to :api
   :name "API calls"}

  {:el :request
   :id :api-database
   :from :api
   :to :database
   :name "SQL queries"}}
```

### 2. Define Views

`models/views.edn`:
```clojure
#{;; Context diagram
  {:el :context-view
   :id :system-context
   :title "System Context"
   :ct #{:user :main-system}}

  ;; Container diagram
  {:el :container-view
   :id :container-view
   :title "Container View"
   :ct #{:webapp :api :database}}}
```

### 3. Generate Diagrams

```bash
overarch -m models/ -r vault/architecture/diagrams/ -f png
```

### 4. Validate

```bash
overarch -m models/ --check
```

## Element Types

| Element | Purpose |
|---------|---------|
| `:person` | Human actor |
| `:system` | Software system |
| `:container` | Deployable unit (app, db, queue) |
| `:component` | Code module within container |
| `:node` | Deployment infrastructure |

## Relationship Types

| Type | Direction |
|------|-----------|
| `:request` | Sync call |
| `:send` | Async message |
| `:publish` | Event publish |
| `:subscribe` | Event consume |
| `:dataflow` | Data movement |

## Reference Documentation

- `references/model-elements.md` - All element types
- `references/relationships.md` - Relationship patterns
- `references/views.md` - View types
- `references/cli-usage.md` - CLI commands

## Success Criteria

```bash
#!/bin/bash
# verify-overarch.sh - Machine-readable verification

verify_overarch() {
  local model_dir="${1:-models}"
  local exit_code=0

  echo "overarch_verification:start"

  # Check model.edn exists
  if [ -f "$model_dir/model.edn" ]; then
    echo "model_edn:exists"
  else
    echo "model_edn:missing"
    exit_code=1
  fi

  # Check views.edn exists
  if [ -f "$model_dir/views.edn" ]; then
    echo "views_edn:exists"
  else
    echo "views_edn:missing"
    exit_code=1
  fi

  # Validate model
  if command -v overarch &>/dev/null; then
    if overarch -m "$model_dir" --check >/dev/null 2>&1; then
      echo "overarch_check:pass"
    else
      echo "overarch_check:fail"
      exit_code=1
    fi
  else
    echo "overarch_check:skipped:not_installed"
  fi

  echo "overarch_verification:exit_code:$exit_code"
  return $exit_code
}

verify_overarch "$@"
```

**Success = all checks pass:**
- [ ] `model_edn:exists` - Model file created
- [ ] `views_edn:exists` - Views file created
- [ ] `overarch_check:pass` - Model validates without errors

## Linking Requirements

When documenting the architecture model:
- Link to ADRs: `Decision: [[decisions/0002-microservices]]`
- Link to arc42 sections: `See [[arc42/05-building-blocks]]`
- Link to threat model: `Threats: [[security/threats]]`

## Related Skills

- `structurizr-diagrams` - Alternative visualization
- `sync-architecture` - Keep models in sync
