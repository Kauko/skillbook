# Skill-Seekers Command Reference

## scrape - Documentation Website Scraping

Scrape documentation websites and generate skills.

```bash
skill-seekers scrape --url <URL> --name <name> [options]
skill-seekers scrape --config <config.json> [options]
```

### Key Options
- `--url URL` - Documentation website URL
- `--name NAME` - Skill name
- `--description DESCRIPTION` - Skill description for frontmatter
- `--config CONFIG` - JSON config file path
- `--async` - Use async scraping (3x faster, 66% less memory)
- `--workers N` - Number of async workers (default: 4, recommended: 8)
- `--skip-scrape` - Skip scraping, use cached data for rebuilding
- `--enhance-local` - AI enhancement using local Claude Code (no API key needed)
- `--dry-run` - Test configuration without scraping

### When to Use
- Creating skills from documentation sites (React, Django, Python docs, etc.)
- When URL structure is consistent and predictable
- For large documentation sets (use `--async --workers 8`)

## github - GitHub Repository Scraping

Scrape GitHub repositories with deep code analysis.

```bash
skill-seekers github --repo <owner/repo> --name <name> [options]
skill-seekers github --config <config.json>
```

### Key Options
- `--repo REPO` - GitHub repository (format: owner/repo)
- `--name NAME` - Skill name
- `--description DESCRIPTION` - Skill description
- `--config CONFIG` - JSON config file path

### Features
- Deep Abstract Syntax Tree (AST) parsing for code analysis
- Automatically includes README, documentation files
- Organizes code by modules/components

### When to Use
- Creating skills from libraries without comprehensive docs
- When you need code examples and implementation details
- For internal tools or company repositories

## pdf - PDF Document Extraction

Extract content from PDF manuals and documentation.

```bash
skill-seekers pdf --pdf <file.pdf> --name <name> [options]
```

### Key Options
- `--pdf FILE` - PDF file path
- `--name NAME` - Skill name
- `--ocr` - Enable OCR for scanned PDFs (requires pytesseract)
- `--extract-tables` - Extract tables from PDF
- `--parallel` - Parallel processing
- `--workers N` - Number of workers for parallel processing
- `--password PASSWORD` - Password for encrypted PDFs

### When to Use
- Converting PDF manuals to skills
- When no web documentation exists
- For proprietary documentation in PDF format

## unified - Multi-Source Scraping

Combine documentation, GitHub, and PDF sources into one skill.

```bash
skill-seekers unified --config <unified-config.json>
```

### Config Structure
```json
{
  "name": "myframework",
  "merge_mode": "rule-based",
  "sources": [
    {
      "type": "documentation",
      "base_url": "https://docs.example.com/",
      "max_pages": 200
    },
    {
      "type": "github",
      "repo": "owner/myframework"
    }
  ]
}
```

### When to Use
- When documentation and code examples are needed together
- To detect conflicts between docs and implementation
- For comprehensive framework skills (e.g., React with docs + repo)

## enhance - AI Enhancement

Enhance generated skills with AI to extract key concepts and examples.

```bash
skill-seekers enhance <skill-directory>
```

### What It Does
- Extracts code examples and patterns
- Identifies key concepts and workflows
- Improves SKILL.md structure and clarity
- Uses local Claude Code (no API key needed)

### When to Use
- After initial scraping to improve quality
- When SKILL.md is too verbose or unstructured
- To extract the most important patterns from large content

## package - Package Skill

Package skill directory into uploadable .skill file.

```bash
skill-seekers package <skill-directory> [options]
```

### Options
- `--no-open` - Don't open output folder after packaging
- `--upload` - Auto-upload to Claude after packaging

### Output
Creates a `.skill` file (zip format) ready for upload to Claude.

## upload - Upload to Claude

Upload packaged skill to Claude.

```bash
skill-seekers upload <skill-file.skill>
```

## estimate - Estimate Scraping Size

Estimate page count and time before scraping.

```bash
skill-seekers estimate <config.json>
```

### When to Use
- Before scraping large documentation sites
- To validate URL patterns and selectors
- To estimate time and resource requirements
