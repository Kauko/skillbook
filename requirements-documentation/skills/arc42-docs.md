---
name: arc42-docs
description: Use when user wants to document system architecture, create architecture overview, or populate arc42 template sections.
requires:
  tools: []
  skills: [init-obsidian-vault, adr-management, iso25010-quality]
---

# arc42 Architecture Documentation

## Prerequisites

```bash
[ -d vault/arc42 ] || { echo "Run init-obsidian-vault first"; exit 1; }
```

### 2. Gather Context

```bash
# Architecture models
find . -name "*.edn" -path "*/models/*" 2>/dev/null | head -5

# Existing ADRs
find vault/decisions/ -name "*.md" 2>/dev/null | head -10

# Threat models
ls vault/security/threagile.yaml 2>/dev/null
```

### 3. Interview User

Ask for:
1. System's primary purpose
2. Main stakeholders
3. Top 3-5 quality goals
4. Major constraints
5. External system interfaces
6. Technology stack decisions

### 4. Create Sections (Parallel Option)

**For faster creation, dispatch parallel subagents for independent sections:**

```
Subagent 1 - Context & Constraints:
  Create 01-introduction.md, 02-constraints.md, 03-context.md

Subagent 2 - Technical Sections:
  Create 05-building-blocks.md, 06-runtime.md, 07-deployment.md

Subagent 3 - Quality & Risk:
  Create 10-quality.md, 11-risks.md, 12-glossary.md
```

**Dependent sections (create after, sequentially):**
- `04-solution.md` - needs context from 01-03
- `08-crosscutting.md` - needs building blocks
- `09-decisions.md` - links to existing ADRs

### Section Quick Reference

| File | Content |
|------|---------|
| `01-introduction.md` | Goals, stakeholders, quality priorities |
| `02-constraints.md` | Technical, organizational, political limits |
| `03-context.md` | System boundary, external interfaces |
| `04-solution.md` | Key decisions, technology choices, patterns |
| `05-building-blocks.md` | Component structure (3 levels) |
| `06-runtime.md` | Key scenarios as sequence diagrams |
| `07-deployment.md` | Infrastructure, environments |
| `08-crosscutting.md` | Logging, security, error handling patterns |
| `09-decisions.md` | ADR summary with links |
| `10-quality.md` | Quality scenarios (ISO 25010) |
| `11-risks.md` | Technical risks and debt |
| `12-glossary.md` | Domain and technical terms |

### 5. Cross-link

Link between sections and to:
- ADRs in `vault/decisions/`
- Requirements in `vault/requirements/`
- Threat model in `vault/security/`

## Reference Documentation

- `references/sections.md` - Detailed guidance for each section
- `references/tips.md` - arc42 best practices and tips

## Success Criteria

- [ ] All 12 arc42 sections created or updated
- [ ] Cross-links between sections working
- [ ] ADRs referenced in section 09
- [ ] Quality scenarios in section 10

## Related Skills

- `init-obsidian-vault` - Creates vault structure (run first)
- `adr-management` - Create and manage ADRs
- `iso25010-quality` - Define quality requirements
- `overarch-modeling` - Architecture model for diagrams
