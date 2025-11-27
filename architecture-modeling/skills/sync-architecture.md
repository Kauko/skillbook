---
description: Use after editing Overarch model to regenerate all derived formats.
---

# Sync Architecture Skill

Use this skill after editing the Overarch model to regenerate all derived formats and ensure consistency across architecture artifacts.

## Purpose

This skill provides an idempotent operation to:
1. Read the canonical Overarch EDN model
2. Regenerate all derived formats (Structurizr DSL, Threagile YAML, etc.)
3. Report what changed
4. Ensure all artifacts are in sync

## Prerequisites Check

Verify required tools are installed:

```bash
# Check Overarch
overarch --version

# Check Structurizr CLI
structurizr-cli --version

# Optional: Check Threagile (for security modeling)
# threagile version
```

Guide installation if missing (see overarch-modeling and structurizr-diagrams skills).

## Core Workflow

### 1. Verify Model Exists

Check that the source model exists:

```bash
if [ ! -f vault/architecture/model.edn ]; then
    echo "ERROR: vault/architecture/model.edn not found"
    echo "Run overarch-modeling skill to create the architecture model first"
    exit 1
fi
```

### 2. Read and Validate Model

Read the Overarch model:

```bash
cat vault/architecture/model.edn
```

Validate EDN syntax:

```bash
# Using Clojure CLI if available
clojure -M -e "(require '[clojure.edn :as edn]) (edn/read-string (slurp \"vault/architecture/model.edn\"))"
```

Or validate structure manually:
- Check balanced brackets
- Verify all IDs are unique
- Ensure relationships reference existing elements

### 3. Capture Pre-Sync State

Take checksums of existing artifacts to detect changes:

```bash
# Create temp directory for comparison
mkdir -p /tmp/architecture-sync

# Backup current state
if [ -f vault/architecture/workspace.dsl ]; then
    cp vault/architecture/workspace.dsl /tmp/architecture-sync/workspace.dsl.old
    md5sum vault/architecture/workspace.dsl > /tmp/architecture-sync/dsl.md5.old
fi

if [ -d vault/architecture/diagrams ]; then
    ls -la vault/architecture/diagrams/ > /tmp/architecture-sync/diagrams.list.old
fi
```

### 4. Regenerate Structurizr DSL

Translate Overarch EDN to Structurizr DSL:

**Translation Process:**
1. Parse EDN model structure
2. Map Overarch elements to Structurizr DSL elements
3. Generate workspace.dsl with views and styles
4. Save to `vault/architecture/workspace.dsl`

**Mapping Reference:**

```
Overarch EDN          → Structurizr DSL
─────────────────────────────────────────
:person              → person
:system              → softwareSystem
:container           → container
:component           → component
:database            → container (shape: cylinder)
:rel                 → relationship (->)
:context-view        → systemContext
:container-view      → container
:component-view      → component
:trust-boundary      → group or boundary
```

**Example Translation:**

Overarch:
```clojure
{:el :system
 :id :my-system
 :name "My System"
 :desc "Description"}
```

Structurizr DSL:
```dsl
mySystem = softwareSystem "My System" "Description"
```

### 5. Generate Architecture Diagrams

Generate visual diagrams using Structurizr CLI:

```bash
# Create diagrams directory
mkdir -p vault/architecture/diagrams

# Generate PNG diagrams
cd vault/architecture
structurizr-cli export -workspace workspace.dsl -format png -output diagrams/

# Also generate SVG for scalability
structurizr-cli export -workspace workspace.dsl -format svg -output diagrams/
```

### 6. Generate Threagile Model (Optional)

If security modeling is needed, generate Threagile YAML:

**Translation Process:**
1. Extract systems, containers, and data assets from Overarch model
2. Map trust boundaries to Threagile trust boundaries
3. Extract data classifications and sensitivity levels
4. Generate `vault/architecture/threagile.yaml`

**Threagile Template:**

```yaml
threagile_version: 1.0.0

title: System Threat Model
author:
  name: Generated from Overarch
  homepage: ""

date: 2025-11-27

management_summary_comment: |
  Generated threat model from Overarch architecture model.

business_overview:
  description: System architecture and security analysis

technical_assets:
  frontend:
    id: frontend
    description: Frontend application
    type: process
    usage: business
    used_as_client_by_human: true
    technologies:
      - web-application
    data_assets_processed:
      - user-data
    communication_links:
      api-connection:
        target: backend
        protocol: https
        authentication: token

  backend:
    id: backend
    description: Backend API service
    type: process
    usage: business
    technologies:
      - service-registry
    data_assets_processed:
      - user-data
      - business-data
    communication_links:
      database-connection:
        target: database
        protocol: binary
        authentication: credentials

  database:
    id: database
    description: Main database
    type: datastore
    usage: business
    technologies:
      - database
    data_assets_stored:
      - user-data
      - business-data

data_assets:
  user-data:
    id: user-data
    description: User information
    usage: business
    quantity: many
    confidentiality: confidential
    integrity: critical
    availability: critical

  business-data:
    id: business-data
    description: Business information
    usage: business
    quantity: many
    confidentiality: strictly-confidential
    integrity: critical
    availability: important

trust_boundaries:
  internet:
    id: internet
    type: network-cloud-provider
    technical_assets_inside:
      - frontend

  internal:
    id: internal
    type: network-dedicated-hoster
    technical_assets_inside:
      - backend
      - database
```

### 7. Update Documentation

Generate or update `vault/architecture/diagrams.md`:

```markdown
# Architecture Diagrams

**Last Updated:** [TIMESTAMP]
**Source Model:** `vault/architecture/model.edn`

## System Context Diagram

Shows the high-level system context with external actors and systems.

![System Context](diagrams/SystemContext.png)

### Elements
- [List key elements from context view]

## Container Diagram

Shows the major containers (applications, services, data stores) within the system.

![Container Diagram](diagrams/Containers.png)

### Containers
- [List containers with brief descriptions]

## Component Diagrams

### [Container Name] Components

Shows the internal structure and components within [Container Name].

![Component Diagram](diagrams/[ContainerName]Components.png)

## Architecture Documentation

- **Model Source:** `vault/architecture/model.edn`
- **Structurizr DSL:** `vault/architecture/workspace.dsl`
- **Threat Model:** `vault/architecture/threagile.yaml` (if applicable)

## Regeneration

To regenerate all architecture artifacts:

1. Edit `vault/architecture/model.edn` (the single source of truth)
2. Run the `sync-architecture` skill
3. Review generated artifacts
4. Commit changes to version control

---

*Generated by architecture-modeling plugin*
```

### 8. Compare and Report Changes

Detect what changed:

```bash
# Compare DSL
if [ -f /tmp/architecture-sync/workspace.dsl.old ]; then
    echo "=== Structurizr DSL Changes ==="
    diff -u /tmp/architecture-sync/workspace.dsl.old vault/architecture/workspace.dsl || true
fi

# List new/updated diagrams
if [ -f /tmp/architecture-sync/diagrams.list.old ]; then
    echo "=== Diagram Changes ==="
    ls -la vault/architecture/diagrams/ > /tmp/architecture-sync/diagrams.list.new
    diff -u /tmp/architecture-sync/diagrams.list.old /tmp/architecture-sync/diagrams.list.new || true
fi

# Summary
echo "=== Sync Summary ==="
echo "✓ Model read from: vault/architecture/model.edn"
echo "✓ DSL generated at: vault/architecture/workspace.dsl"
echo "✓ Diagrams generated in: vault/architecture/diagrams/"
echo "✓ Documentation updated: vault/architecture/diagrams.md"
```

### 9. Validate Generated Artifacts

Verify all artifacts are valid:

```bash
# Validate Structurizr DSL
structurizr-cli validate -workspace vault/architecture/workspace.dsl

# Check diagrams exist
required_diagrams=("SystemContext.png" "Containers.png")
for diagram in "${required_diagrams[@]}"; do
    if [ -f "vault/architecture/diagrams/$diagram" ]; then
        echo "✓ $diagram exists"
    else
        echo "✗ $diagram missing"
    fi
done

# Validate Threagile YAML (if exists)
if [ -f vault/architecture/threagile.yaml ]; then
    # threagile analyze -model vault/architecture/threagile.yaml -dry-run
    echo "✓ Threagile model exists"
fi
```

## Idempotency

This operation is idempotent - running it multiple times without model changes produces identical output:

**Test Idempotency:**

```bash
# Run sync twice
echo "First sync..."
# [sync operations]
cp vault/architecture/workspace.dsl /tmp/sync1.dsl

echo "Second sync (should be identical)..."
# [sync operations]
cp vault/architecture/workspace.dsl /tmp/sync2.dsl

# Compare
diff /tmp/sync1.dsl /tmp/sync2.dsl
if [ $? -eq 0 ]; then
    echo "✓ Sync is idempotent"
else
    echo "✗ Sync produced different output"
fi
```

## Change Detection

Automatically detect what triggered the sync:

```bash
# Check if model was updated
model_time=$(stat -f %m vault/architecture/model.edn)
dsl_time=$(stat -f %m vault/architecture/workspace.dsl 2>/dev/null || echo 0)

if [ $model_time -gt $dsl_time ]; then
    echo "Model updated since last sync (reason for regeneration)"
    echo "Model modified: $(stat -f %Sm vault/architecture/model.edn)"
fi

# List recently modified model sections
echo "Recent model changes:"
git log -1 --pretty=format:"%h - %s (%ar)" -- vault/architecture/model.edn
```

## Artifacts Inventory

This skill generates/updates:

| Artifact | Path | Format | Purpose |
|----------|------|--------|---------|
| Source Model | `vault/architecture/model.edn` | Overarch EDN | Single source of truth |
| Structurizr DSL | `vault/architecture/workspace.dsl` | DSL | Structurizr workspace |
| System Context | `vault/architecture/diagrams/SystemContext.png` | PNG | Context diagram |
| Container Diagram | `vault/architecture/diagrams/Containers.png` | PNG | Container diagram |
| Component Diagrams | `vault/architecture/diagrams/*Components.png` | PNG | Component diagrams |
| Documentation | `vault/architecture/diagrams.md` | Markdown | Embedded diagrams |
| Threat Model | `vault/architecture/threagile.yaml` | YAML | Security analysis (optional) |

## Error Handling

Handle common errors gracefully:

**Model Not Found:**
```bash
if [ ! -f vault/architecture/model.edn ]; then
    echo "ERROR: Architecture model not found"
    echo "Create model using overarch-modeling skill first"
    exit 1
fi
```

**Invalid EDN:**
```bash
# Attempt to parse
if ! clojure -M -e "(edn/read-string (slurp \"vault/architecture/model.edn\"))" 2>/dev/null; then
    echo "ERROR: Invalid EDN syntax in model.edn"
    echo "Check brackets, keywords, and quotes"
    exit 1
fi
```

**Missing Tools:**
```bash
if ! command -v structurizr-cli &> /dev/null; then
    echo "ERROR: structurizr-cli not found"
    echo "Install: brew install structurizr-cli"
    exit 1
fi
```

**Generation Failures:**
```bash
if ! structurizr-cli export -workspace workspace.dsl -format png -output diagrams/; then
    echo "ERROR: Failed to generate diagrams"
    echo "Check workspace.dsl syntax"
    structurizr-cli validate -workspace workspace.dsl
    exit 1
fi
```

## Advanced: Selective Regeneration

For large models, allow selective regeneration:

```bash
# Regenerate only DSL
sync_dsl_only() {
    echo "Regenerating DSL only..."
    # [translate EDN to DSL]
    echo "✓ DSL updated"
}

# Regenerate only diagrams
sync_diagrams_only() {
    echo "Regenerating diagrams only..."
    cd vault/architecture
    structurizr-cli export -workspace workspace.dsl -format png -output diagrams/
    echo "✓ Diagrams updated"
}

# Regenerate only specific view
sync_view() {
    view_name=$1
    echo "Regenerating view: $view_name"
    # [generate specific view]
}
```

## Integration with Version Control

After sync, prepare for commit:

```bash
# Show what changed
echo "=== Git Status ==="
git status vault/architecture/

# Show diff summary
echo "=== Changes Summary ==="
git diff --stat vault/architecture/

# Suggest commit message
echo "Suggested commit message:"
echo "docs(architecture): regenerate diagrams from updated model"
echo ""
echo "- Updated Structurizr DSL"
echo "- Regenerated architecture diagrams"
echo "- Updated documentation"
```

## Output Format

After sync, provide structured report:

```
╔════════════════════════════════════════╗
║  Architecture Sync Complete            ║
╚════════════════════════════════════════╝

📋 Source Model
   └─ vault/architecture/model.edn
      Modified: 2025-11-27 10:30:00

📝 Generated DSL
   └─ vault/architecture/workspace.dsl
      Status: ✓ Updated
      Changes: 15 lines added, 3 lines removed

🖼️  Generated Diagrams
   ├─ SystemContext.png ✓
   ├─ Containers.png ✓
   └─ BackendComponents.png ✓

📄 Documentation
   └─ diagrams.md ✓ Updated

🔒 Threat Model (optional)
   └─ threagile.yaml ✓ Generated

═══════════════════════════════════════

Summary:
- 1 model file read
- 1 DSL file generated
- 3 diagram files generated
- 1 documentation file updated
- 0 errors, 0 warnings

Next steps:
1. Review generated artifacts
2. Test diagrams render correctly
3. Commit changes: git add vault/architecture && git commit
```

## Best Practices

1. **Run After Every Model Edit**: Always sync after changing model.edn
2. **Review Before Commit**: Check generated artifacts before committing
3. **Commit Atomically**: Commit model and derived artifacts together
4. **Descriptive Commits**: Explain what changed in the model, not just "regenerated"
5. **CI Integration**: Add sync check to CI to ensure artifacts are up-to-date
6. **Backup Before Sync**: Keep backups during development
7. **Validate Post-Sync**: Always validate generated artifacts
8. **Document Changes**: Update diagrams.md with change notes

## Automation

Create a git pre-commit hook:

```bash
#!/bin/bash
# .git/hooks/pre-commit

# Check if model.edn changed
if git diff --cached --name-only | grep -q "vault/architecture/model.edn"; then
    echo "Architecture model changed, checking derived artifacts..."

    # Check if DSL is up-to-date
    if [ vault/architecture/model.edn -nt vault/architecture/workspace.dsl ]; then
        echo "ERROR: model.edn changed but workspace.dsl not regenerated"
        echo "Run: sync-architecture skill"
        exit 1
    fi

    echo "✓ Architecture artifacts up-to-date"
fi
```

## CI/CD Integration

Add to CI pipeline:

```yaml
# .github/workflows/architecture.yml
name: Architecture Validation

on:
  pull_request:
    paths:
      - 'vault/architecture/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install Structurizr CLI
        run: |
          curl -L https://github.com/structurizr/cli/releases/latest/download/structurizr-cli.zip -o structurizr-cli.zip
          unzip structurizr-cli.zip
          sudo mv structurizr-cli /usr/local/bin/

      - name: Validate DSL
        run: structurizr-cli validate -workspace vault/architecture/workspace.dsl

      - name: Check sync status
        run: |
          # Verify model timestamp vs derived artifacts
          if [ vault/architecture/model.edn -nt vault/architecture/workspace.dsl ]; then
            echo "ERROR: Derived artifacts out of sync"
            exit 1
          fi
```

## Example Usage

**User Request:** "I updated the architecture model, sync everything"

**Assistant Actions:**
1. Read `vault/architecture/model.edn`
2. Validate EDN syntax
3. Capture current state for comparison
4. Regenerate Structurizr DSL
5. Generate diagrams (PNG/SVG)
6. Generate Threagile YAML (if applicable)
7. Update `diagrams.md`
8. Compare old vs new artifacts
9. Report changes with diff summary
10. Suggest git commit

**User Request:** "Check if my architecture artifacts are in sync"

**Assistant Actions:**
1. Compare timestamps: model.edn vs workspace.dsl vs diagrams
2. Report staleness if found
3. If stale: offer to run full sync
4. If current: confirm all artifacts up-to-date

---

Remember: This skill is idempotent and safe to run multiple times. It ensures all architecture artifacts stay in sync with the canonical Overarch model.
