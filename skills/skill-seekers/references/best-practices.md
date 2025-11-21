# Skill-Seekers Best Practices

## Choosing the Right Source Type

### Documentation-Only (`scrape`)
**Use when:**
- Official documentation site exists
- Documentation is comprehensive and up-to-date
- URL structure is consistent

**Example use cases:**
- React, Vue, Django documentation
- API reference sites
- Framework guides

**Command pattern:**
```bash
skill-seekers scrape --url https://docs.example.com --name library-name --async --workers 8
```

### GitHub-Only (`github`)
**Use when:**
- No comprehensive documentation exists
- Code examples are more valuable than docs
- Working with internal tools or libraries
- Documentation is outdated but code is current

**Example use cases:**
- Small libraries with minimal docs
- Internal company tools
- Open-source projects with poor documentation

**Command pattern:**
```bash
skill-seekers github --repo owner/repo-name --name library-name
```

### Multi-Source (`unified`)
**Use when:**
- Both documentation and code analysis are valuable
- Documentation might be outdated (GitHub provides truth)
- Framework has docs + extensive example repos

**Example use cases:**
- Major frameworks (React, Django, FastAPI)
- Libraries with docs + rich example repositories
- Comprehensive skills needing both conceptual and implementation knowledge

**Command pattern:**
```bash
# Create unified config first, then:
skill-seekers unified --config configs/library-unified.json
```

## Performance Optimization

### For Large Documentation (1000+ pages)
1. **Always use async mode:**
   ```bash
   --async --workers 8
   ```
   - 3x faster than synchronous
   - 66% less memory usage
   - Handles rate limiting better

2. **Enable checkpoints for 10K+ pages:**
   ```json
   {
     "checkpoint": {
       "enabled": true,
       "interval": 1000
     }
   }
   ```

3. **Test with limited pages first:**
   ```json
   {
     "max_pages": 20
   }
   ```
   Then increase after validating selectors work.

### For Small Documentation (<100 pages)
- Synchronous mode is fine
- No need for checkpoints
- Consider if `llms.txt` exists (10x faster if available)

## Config File Strategy

### Start Simple
Don't create config files initially. Use command-line flags:
```bash
skill-seekers scrape --url https://docs.library.com --name library
```

### Create Config When Needed
Save to config file when you need:
- Custom CSS selectors
- URL pattern filtering
- Category organization
- Checkpoint configuration
- Reusable builds (--skip-scrape)

### Config File Templates
Use asset templates in this skill as starting points, then customize.

## Common Workflow Patterns

### Quick One-Off Skill (Documentation)
```bash
# 1. Scrape with async
skill-seekers scrape --url https://docs.library.com --name library --async --workers 8

# 2. Enhance
skill-seekers enhance output/library/

# 3. Package
skill-seekers package output/library/
```

### GitHub Library Without Docs
```bash
# 1. Scrape repo
skill-seekers github --repo owner/library --name library

# 2. Enhance
skill-seekers enhance output/library/

# 3. Package
skill-seekers package output/library/
```

### Comprehensive Framework Skill
```bash
# 1. Create unified config (copy from assets/config-templates/unified.json)
# 2. Customize config with docs URL + GitHub repo
# 3. Run unified scrape
skill-seekers unified --config library-unified.json

# 4. Enhance
skill-seekers enhance output/library/

# 5. Package
skill-seekers package output/library/
```

## Enhancement Strategy

### Always Enhance When
- Scraped content is verbose or repetitive
- Need to extract key patterns from large docs
- SKILL.md structure needs improvement

### Skip Enhancement When
- Documentation is already well-structured
- Content is concise
- Time is critical (enhancement adds 5-10 minutes)

### Enhancement Tips
- Enhancement uses local Claude Code (no API key needed)
- Improves SKILL.md quality significantly
- Extracts code examples and patterns automatically
- Worth the extra time for most skills

## Troubleshooting

### No Content Scraped
**Problem:** CSS selectors don't match page structure

**Solution:**
1. Open docs site in browser
2. Use DevTools to inspect content elements
3. Find the right selectors (usually `article`, `main`, `.content`)
4. Create config file with custom selectors:
   ```json
   {
     "selectors": {
       "main_content": "article.docs-content",
       "title": "h1.page-title"
     }
   }
   ```

### Rate Limited / Blocked
**Problem:** Too many requests too quickly

**Solution:**
1. Add rate limiting to config:
   ```json
   {
     "rate_limit": 0.5
   }
   ```
2. Or use async mode (handles rate limiting better):
   ```bash
   --async --workers 8
   ```

### Memory Issues
**Problem:** Running out of memory on large docs

**Solution:**
1. Switch to async mode (66% less memory)
2. Reduce number of workers
3. Enable checkpoints for crash recovery

### Cached Data Stale
**Problem:** Changes not appearing in rebuilt skill

**Solution:**
Delete cached data and re-scrape:
```bash
rm -rf output/library_data/
skill-seekers scrape --config library.json
```

Or force fresh scrape:
```bash
skill-seekers scrape --config library.json --fresh
```

## Quality Checklist

Before finalizing a skill, verify:

- [ ] SKILL.md frontmatter has clear, comprehensive description
- [ ] Content is well-organized (not just a dump of scraped pages)
- [ ] Code examples are present and accurate
- [ ] Key concepts are highlighted
- [ ] Unnecessary content is removed
- [ ] Enhancement was run (if applicable)
- [ ] Skill has been tested with a few real queries
