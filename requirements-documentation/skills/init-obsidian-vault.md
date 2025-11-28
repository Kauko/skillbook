---
name: init-obsidian-vault
description: Use when starting a new project, setting up documentation structure, or before using other vault-based skills like arc42-docs.
requires:
  tools: []
  skills: []
---

# Initialize Obsidian Vault

## Prerequisites

```bash
# No external tools required - creates directories and files
ls -la vault/ 2>/dev/null || echo "No vault - will create"
```

Idempotent: only creates missing directories/files.

### 2. Create Structure

```bash
mkdir -p vault/{architecture,requirements,specifications,decisions,security,arc42,templates}
```

### 3. Create README Files

**vault/README.md:**
```markdown
# Project Documentation

## Navigation

- [[architecture/README|Architecture]]
- [[requirements/README|Requirements]]
- [[specifications/README|Specifications]]
- [[decisions/README|Decisions (ADRs)]]
- [[security/README|Security]]
- [[arc42/README|arc42 Documentation]]
```

**vault/decisions/README.md:**
```markdown
# Architectural Decision Records

See [[index]] for complete list.

## Create New ADR

Use adr-management skill or copy template.
```

### 4. Create Templates

**vault/templates/adr-template.md:**
```markdown
# NNNN. Title

**Date**: {{date}}
**Status**: Proposed

## Context
[...]

## Decision
[...]

## Consequences
[...]
```

### 5. Configure .obsidian

```bash
mkdir -p vault/.obsidian
```

Create `vault/.obsidian/app.json`:
```json
{
  "useMarkdownLinks": false,
  "newFileLocation": "current"
}
```

## Final Structure

```
vault/
├── README.md
├── architecture/
├── requirements/
├── specifications/
├── decisions/
│   ├── README.md
│   └── index.md
├── security/
├── arc42/
└── templates/
    ├── adr-template.md
    ├── meeting-notes.md
    └── requirement.md
```

## Output

```
Vault initialized at vault/

Created directories:
✓ vault/architecture
✓ vault/requirements
✓ vault/specifications
✓ vault/decisions
✓ vault/security
✓ vault/arc42
✓ vault/templates

Next: Use arc42-docs, adr-management, or other skills
```

## Success Criteria

- [ ] `vault/` directory exists with subdirectories
- [ ] `vault/README.md` created
- [ ] `vault/decisions/index.md` created
- [ ] Template files in `vault/templates/`

## Related Skills

- `arc42-docs` - Architecture documentation
- `adr-management` - Create ADRs
- `iso25010-quality` - Quality requirements
