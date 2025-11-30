---
name: deepwiki-lookup
description: Use when documentation is missing or unclear. Query deepwiki MCP for library docs, API references, and examples.
requires:
  tools: []
  mcps: [deepwiki]
  skills: []
skip_when:
  - Documentation is already known
  - deepwiki MCP not installed
---

# Deepwiki Lookup

Use deepwiki MCP to find documentation when information is missing.

## Prerequisites

Deepwiki MCP must be installed. If not available, fall back to:
1. Web search
2. Reading source code
3. Asking user

## When to Use

- Unknown library API
- Unclear function behavior
- Missing configuration options
- Need examples for a library
- Version-specific documentation

## How to Query

### Library documentation

Query for specific library:

```
Query deepwiki for: "malli schema validation examples"
Query deepwiki for: "reitit routing configuration"
Query deepwiki for: "shadow-cljs hot reload setup"
```

### API reference

Query for specific functions:

```
Query deepwiki for: "malli/validate function signature"
Query deepwiki for: "reitit.ring/router options"
```

### Configuration

Query for config options:

```
Query deepwiki for: "shadow-cljs.edn configuration options"
Query deepwiki for: "clj-kondo config.edn linters"
```

## Integrating Results

When you find useful documentation:

1. **Use it** - Apply the information to your task
2. **Note the source** - Mention where you found it
3. **Update local docs if helpful** - Add to project documentation

## Fallbacks

If deepwiki MCP unavailable:

1. **Web search** - Use WebSearch tool
2. **Source code** - Read the library source
3. **Ask user** - They may know or have docs

## Success Criteria

- [ ] Found needed documentation
- [ ] Applied information correctly
- [ ] Noted source for reference
