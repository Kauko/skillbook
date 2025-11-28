---
name: skill-seekers
description: Use when user asks to create a skill from docs, scrape documentation into a skill, or package a library's reference into Claude Code skill format.
requires:
  tools: [skill-seekers]
  skills: []
---

# Skill Seekers

Create Claude skills from documentation websites, GitHub repos, and PDFs.

## Prerequisites

```bash
command -v skill-seekers >/dev/null || { echo "Install skill-seekers CLI"; exit 1; }
```

## Decision Tree

| Source | Command |
|--------|---------|
| Has comprehensive docs | `scrape` |
| Has docs + GitHub repo | `unified` |
| Only GitHub (no docs) | `github` |
| PDF documentation | `pdf` |

## Quick Commands

### Documentation

```bash
skill-seekers scrape \
  --url https://docs.library.com \
  --name library-name \
  --description "Use when..." \
  --async --workers 8
```

### GitHub Only

```bash
skill-seekers github \
  --repo owner/repo-name \
  --name library-name \
  --description "Use when..."
```

### Docs + GitHub (Unified)

Copy `assets/config-templates/unified.json`, customize, then:
```bash
skill-seekers unified --config library-unified.json
```

### PDF

```bash
skill-seekers pdf --file path/to/doc.pdf --name library-name
```

## Standard Workflow

```bash
# 1. Scrape
skill-seekers [scrape|github|unified] ...

# 2. Enhance (always do this)
skill-seekers enhance output/library-name/

# 3. Package
skill-seekers package output/library-name/
```

Output: `library-name.skill` file for user to install.

## Config File (When Needed)

Use config file for:
- Custom CSS selectors
- URL pattern filtering
- Large docs (10K+ pages)

Templates in `assets/config-templates/`:
- `docs-only.json`
- `unified.json`
- `github-only.json`

## Performance

- **Always use async:** `--async --workers 8` (3x faster)
- **Test small first:** Add `"max_pages": 20` to config
- **Check for llms.txt:** 10x faster if site has it

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No content | Check CSS selectors in DevTools |
| Rate limited | Add `"rate_limit": 0.5"` to config |
| Memory issues | Use `--async`, reduce workers |

## Example

**User: "Create a skill for FastAPI"**

```bash
# FastAPI has docs + GitHub, use unified
# 1. Customize assets/config-templates/unified.json
skill-seekers unified --config fastapi.json
skill-seekers enhance output/fastapi/
skill-seekers package output/fastapi/
```

## Success Criteria

- [ ] Content scraped/extracted from source
- [ ] `skill-seekers enhance` completed
- [ ] `.skill` package created
- [ ] Skill installable by user

## References

- `references/command-reference.md` - All options
- `references/best-practices.md` - Strategies
- `assets/config-templates/` - Config templates
