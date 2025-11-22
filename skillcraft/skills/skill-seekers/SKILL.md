---
name: skill-seekers
description: Use when the user asks you to create a new skill from external sources (documentation websites, GitHub repositories, or PDF files). Automates the process of scraping, organizing, enhancing, and packaging content into Claude skills. Use for requests like "create a skill for [library name]", "make a skill from [docs URL]", or "turn [GitHub repo] into a skill".
---

# Skill Seekers

Create Claude skills automatically from documentation websites, GitHub repositories, and PDF files.

## Overview

When the user asks you to create a skill from external sources, use skill-seekers to:
1. Determine the best source type (docs, GitHub, or unified)
2. Scrape and organize the content
3. Enhance the skill with AI
4. Package it for Claude

## Workflow

### Step 1: Analyze the Request

Determine what sources are available:

```bash
# Check for documentation site
# Look for official docs URL (e.g., https://docs.library.com, https://library.dev/docs)

# Check for GitHub repository
# Format: owner/repo-name
```

**Decision tree:**

- **Has comprehensive docs** → Use `scrape` command
- **Has docs + important GitHub repo** → Use `unified` command
- **Has minimal/no docs, only GitHub** → Use `github` command
- **Has PDF documentation** → Use `pdf` command

### Step 2: Run the Scraping

#### For Documentation Only

**Simple approach (no config file):**
```bash
skill-seekers scrape \
  --url https://docs.library.com \
  --name library-name \
  --description "Brief description of when to use this skill" \
  --async --workers 8
```

**With config file (for complex requirements):**
1. Copy template from `assets/config-templates/docs-only.json`
2. Customize: name, description, base_url, selectors, url_patterns
3. Save as `library-name.json`
4. Run:
```bash
skill-seekers scrape --config library-name.json --async --workers 8
```

**When to use config file:**
- Need custom CSS selectors
- URL pattern filtering required
- Large docs (10K+ pages) needing checkpoints
- Want to organize content by categories

#### For GitHub Repository Only

```bash
skill-seekers github \
  --repo owner/repo-name \
  --name library-name \
  --description "Brief description of when to use this skill"
```

#### For Combined Sources (Unified)

1. Copy template from `assets/config-templates/unified.json`
2. Customize:
   - `name` and `description`
   - Documentation `base_url`
   - GitHub `repo` (format: owner/repo-name)
   - Adjust `max_pages` and `max_files` as needed
3. Save as `library-name-unified.json`
4. Run:
```bash
skill-seekers unified --config library-name-unified.json
```

**Use unified when:**
- Both docs and code examples are valuable
- Docs might be outdated (GitHub provides truth)
- Creating comprehensive framework skills

### Step 3: Enhance the Skill

Always enhance unless time is critical:

```bash
skill-seekers enhance output/library-name/
```

**What enhancement does:**
- Extracts code examples and patterns
- Identifies key concepts
- Improves SKILL.md structure
- Uses local Claude Code (no API key needed)

**Time cost:** 5-10 minutes extra

### Step 4: Package the Skill

```bash
skill-seekers package output/library-name/
```

This creates `library-name.skill` file ready for the user to install.

### Step 5: Deliver to User

The packaged `.skill` file will be in the output directory. Inform the user:
- Where the file is located
- How to install it (they can drag-and-drop to Claude or use upload command)
- What the skill enables them to do

## Quick Reference

### Common Patterns

**Small library with docs:**
```bash
skill-seekers scrape --url https://docs.library.com --name library --async --workers 8
skill-seekers enhance output/library/
skill-seekers package output/library/
```

**GitHub repo without docs:**
```bash
skill-seekers github --repo owner/library --name library
skill-seekers enhance output/library/
skill-seekers package output/library/
```

**Major framework (docs + GitHub):**
```bash
# 1. Copy assets/config-templates/unified.json
# 2. Customize and save as library-unified.json
skill-seekers unified --config library-unified.json
skill-seekers enhance output/library/
skill-seekers package output/library/
```

### Performance Tips

- **Always use async for docs:** `--async --workers 8` (3x faster, 66% less memory)
- **Test first with small page limit:** Add `"max_pages": 20` to config, then increase
- **Check for llms.txt:** If site has `llms.txt` or `llms-full.txt`, scraping is 10x faster
- **Enable checkpoints for huge docs:** Add checkpoint config for 10K+ pages

### Troubleshooting

**No content scraped:**
- Check CSS selectors with browser DevTools
- Create config file with custom selectors (see `references/best-practices.md`)

**Rate limited:**
- Add `"rate_limit": 0.5` to config
- Async mode handles rate limiting better

**Memory issues:**
- Use `--async` (66% less memory)
- Reduce workers or enable checkpoints

## Detailed References

- **Command options and flags:** See `references/command-reference.md`
- **Best practices and strategies:** See `references/best-practices.md`
- **Config templates:** Available in `assets/config-templates/`

## Example Scenarios

**User: "Create a skill for Malli"**
1. Research: Malli is a Clojure library with GitHub repo (metosin/malli)
2. Check for docs: Has documentation at https://github.com/metosin/malli (in README)
3. Decision: Use `github` (docs are in repo)
4. Execute:
```bash
skill-seekers github --repo metosin/malli --name malli \
  --description "Use when working with Malli, a Clojure data validation library"
skill-seekers enhance output/malli/
skill-seekers package output/malli/
```

**User: "Make a skill from React documentation"**
1. Research: React has docs at https://react.dev
2. Decision: Use `scrape` with async
3. Execute:
```bash
skill-seekers scrape --url https://react.dev --name react \
  --description "Use when building React applications" \
  --async --workers 8
skill-seekers enhance output/react/
skill-seekers package output/react/
```

**User: "Create a comprehensive FastAPI skill"**
1. Research: FastAPI has docs (https://fastapi.tiangolo.com) + GitHub (tiangolo/fastapi)
2. Decision: Use `unified` for comprehensive coverage
3. Copy `assets/config-templates/unified.json`
4. Customize and save as `fastapi-unified.json`:
```json
{
  "name": "fastapi",
  "description": "Use when building FastAPI applications",
  "merge_mode": "rule-based",
  "sources": [
    {
      "type": "documentation",
      "base_url": "https://fastapi.tiangolo.com/",
      "max_pages": 200
    },
    {
      "type": "github",
      "repo": "tiangolo/fastapi"
    }
  ]
}
```
5. Execute:
```bash
skill-seekers unified --config fastapi-unified.json
skill-seekers enhance output/fastapi/
skill-seekers package output/fastapi/
```
