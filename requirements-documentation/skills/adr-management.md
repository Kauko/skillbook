---
name: adr-management
description: Use when making ANY architectural decision that affects system structure, is difficult to reverse, or sets precedent. Create ADR BEFORE implementing the decision.
requires:
  tools: []
  skills: [init-obsidian-vault]
skip_when:
  - Decision is easily reversible (e.g., variable naming, minor refactoring)
  - Decision is local to a single function/file with no broader impact
  - User explicitly states this is exploratory/throwaway work
---

# ADR Management

## When to Create an ADR

- Affects system structure/architecture
- Difficult or expensive to reverse
- Impacts multiple components or teams
- Sets precedent for future decisions
- Has significant trade-offs

## Workflow

### 1. Find Next Number

```bash
ls vault/decisions/ | grep -E "^[0-9]{4}-" | sort -r | head -1
# Increment the number (start with 0001 if empty)
```

### 2. Create ADR

`vault/decisions/NNNN-title-with-dashes.md`:

```markdown
# NNNN. Decision Title

**Date**: YYYY-MM-DD
**Status**: Proposed | Accepted | Superseded | Deprecated

## Context

[What situation requires a decision? What forces are at play?]

## Decision

[What is the change being proposed/made?]

## Consequences

**Positive:**
- [Benefit 1]
- [Benefit 2]

**Negative:**
- [Drawback 1]
- [Drawback 2]

## Alternatives Considered

### Alternative A
- Pros: [...]
- Cons: [...]
- Why rejected: [...]

## Related

- Supersedes: [[NNNN-previous]]
- Related: [[NNNN-other]]
- Implements: [[requirement-X]]
```

### 3. Update Index

Add to `vault/decisions/index.md`:

```markdown
| # | Title | Status | Date |
|---|-------|--------|------|
| 0005 | [[0005-use-postgresql]] | Accepted | 2025-01-15 |
```

### 4. Cross-link

Reference ADR from:
- `vault/arc42/09-decisions.md`
- Related component documentation
- Quality requirements it addresses

## Status Transitions

```
Proposed → Accepted → Superseded (by new ADR)
                   → Deprecated (no longer relevant)
```

When superseding, update old ADR:
```markdown
**Status**: Superseded by [[NNNN-new-decision]]
```

## Reference Documentation

- `references/templates.md` - Additional ADR templates
- `references/best-practices.md` - Writing effective ADRs

## Success Criteria

```bash
#!/bin/bash
# verify-adr.sh - Machine-readable ADR verification

verify_adr() {
  local adr_file="$1"
  local vault_dir="${2:-vault}"
  local exit_code=0

  echo "adr_verification:start"
  echo "adr_file:$adr_file"

  # Check ADR file exists
  if [ -f "$adr_file" ]; then
    echo "adr_exists:true"
  else
    echo "adr_exists:false"
    echo "adr_verification:exit_code:1"
    return 1
  fi

  # Check required sections
  for section in "Context" "Decision" "Consequences"; do
    if grep -q "^## $section" "$adr_file"; then
      echo "section_${section,,}:present"
    else
      echo "section_${section,,}:missing"
      exit_code=1
    fi
  done

  # Check status field
  if grep -q "^\*\*Status\*\*:" "$adr_file"; then
    local status
    status=$(grep "^\*\*Status\*\*:" "$adr_file" | sed 's/.*: //')
    echo "status:$status"
  else
    echo "status:missing"
    exit_code=1
  fi

  # Check index updated
  local adr_basename
  adr_basename=$(basename "$adr_file" .md)
  if [ -f "$vault_dir/decisions/index.md" ]; then
    if grep -q "$adr_basename" "$vault_dir/decisions/index.md"; then
      echo "index_updated:true"
    else
      echo "index_updated:false"
      exit_code=1
    fi
  else
    echo "index_updated:no_index_file"
  fi

  echo "adr_verification:exit_code:$exit_code"
  return $exit_code
}

verify_adr "$@"
```

**Success = all checks pass:**
- [ ] `adr_exists:true` - ADR file created
- [ ] `section_context:present` - Context section exists
- [ ] `section_decision:present` - Decision section exists
- [ ] `section_consequences:present` - Consequences section exists
- [ ] `status:Proposed|Accepted` - Valid status field
- [ ] `index_updated:true` - Index references this ADR

## Related Skills

- `arc42-docs` - Section 9 summarizes ADRs
- `iso25010-quality` - Quality-driven decisions
