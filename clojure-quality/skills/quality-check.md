---
name: quality-check
description: Use AFTER writing or modifying any Clojure code (*.clj, *.cljs, *.cljc, *.edn). Run before committing. Do not skip even for "small changes."
requires:
  tools: []  # Dynamically detects installed tools
  skills: [clojure-style]
skip_when:
  - Only reading or exploring code (no modifications made)
  - Working exclusively on non-Clojure files (markdown, yaml, etc.)
  - User explicitly requests to skip with stated reason
  - Running in a context where Clojure tooling is unavailable (e.g., CI without tools)
---

# Clojure Quality Check

Run all configured quality tools. Automatically detects which tools are installed.

## Trigger Conditions

Run this skill when:
- ANY `.clj`, `.cljs`, `.cljc`, or `.edn` file is modified
- Before ANY git commit containing Clojure changes
- User asks to "lint", "check", "format", or "review" code
- After generating new Clojure code

## Tool Detection

First, detect available tools:

```bash
#!/bin/bash
# quality-detect.sh - Detect available quality tools

echo "tool_detection:start"

# clj-kondo: semantic analysis
if command -v clj-kondo &>/dev/null; then
  echo "clj-kondo:available"
  [ -d .clj-kondo ] && echo "clj-kondo:configured" || echo "clj-kondo:unconfigured"
else
  echo "clj-kondo:not_installed"
fi

# splint: idiom linting
if command -v splint &>/dev/null; then
  echo "splint:available"
  [ -f .splint.edn ] && echo "splint:configured" || echo "splint:unconfigured"
else
  echo "splint:not_installed"
fi

# cljfmt: formatting
if command -v clojure &>/dev/null; then
  if clojure -Ttools list 2>/dev/null | grep -q cljfmt; then
    echo "cljfmt:available"
    [ -f .cljfmt.edn ] && echo "cljfmt:configured" || echo "cljfmt:unconfigured"
  else
    echo "cljfmt:not_installed"
  fi
else
  echo "clojure:not_installed"
fi

echo "tool_detection:complete"
```

## Workflow

### 1. Run Available Tools

**Order: format first, then lint** (linters should check formatted code).

```bash
#!/bin/bash
# quality-check.sh - Run all available quality tools

set -euo pipefail
RESULTS_FILE="/tmp/claude/quality-results.txt"
> "$RESULTS_FILE"

log_result() {
  echo "$1" | tee -a "$RESULTS_FILE"
}

log_result "quality_check:start:$(date -Iseconds)"
FAILED_TOOLS=""

# 1. Formatting (cljfmt) - runs first
if command -v clojure &>/dev/null && clojure -Ttools list 2>/dev/null | grep -q cljfmt; then
  log_result "cljfmt:running"
  if clojure -Tcljfmt check 2>&1; then
    log_result "cljfmt:pass"
  else
    log_result "cljfmt:fail"
    FAILED_TOOLS="$FAILED_TOOLS cljfmt"
  fi
else
  log_result "cljfmt:skipped:not_available"
fi

# 2. Semantic analysis (clj-kondo)
if command -v clj-kondo &>/dev/null; then
  log_result "clj-kondo:running"
  if clj-kondo --lint src test 2>&1; then
    log_result "clj-kondo:pass"
  else
    log_result "clj-kondo:fail"
    FAILED_TOOLS="$FAILED_TOOLS clj-kondo"
  fi
else
  log_result "clj-kondo:skipped:not_available"
fi

# 3. Idiom linting (splint)
if command -v splint &>/dev/null; then
  log_result "splint:running"
  if splint --check src test 2>&1; then
    log_result "splint:pass"
  else
    log_result "splint:fail"
    FAILED_TOOLS="$FAILED_TOOLS splint"
  fi
else
  log_result "splint:skipped:not_available"
fi

# Summary
if [ -z "$FAILED_TOOLS" ]; then
  log_result "quality_check:pass"
  exit 0
else
  log_result "quality_check:fail:$FAILED_TOOLS"
  exit 1
fi
```

### 2. Fix Issues

**Auto-fixable (run these first):**
```bash
# Fix formatting
clojure -Tcljfmt fix

# Fix idioms
splint --fix src test
```

**Manual fixes required:**
```bash
# clj-kondo errors need manual fixes - get structured output
clj-kondo --lint src test --config '{:output {:format :edn}}' > /tmp/claude/kondo-errors.edn
```

### 3. Verify Fixes

Re-run the quality check after fixes.

## Tool Comparison

| Tool | Analyzes | Auto-fix | Exit Code |
|------|----------|----------|-----------|
| **cljfmt** | Formatting | Yes | 0=pass, 1=issues |
| **clj-kondo** | Semantics | No | 0=pass, 2=warnings, 3=errors |
| **splint** | Idioms | Partial | 0=pass, 1=issues |

## Machine-Readable Output

```bash
# Get structured output for programmatic processing

# clj-kondo EDN output
clj-kondo --lint src test --config '{:output {:format :edn}}' 2>/dev/null

# splint EDN output
splint --format edn src test 2>/dev/null

# Combined quality results
cat /tmp/claude/quality-results.txt
```

**Output format:**
```
quality_check:start:2024-01-15T10:30:00+00:00
cljfmt:pass
clj-kondo:fail
splint:pass
quality_check:fail: clj-kondo
```

## Common Issues Quick Reference

| Tool | Error | Fix |
|------|-------|-----|
| cljfmt | Wrong indentation | `clojure -Tcljfmt fix` |
| clj-kondo | Unresolved symbol | Add require or fix typo |
| clj-kondo | Wrong arity | Check function signature |
| clj-kondo | Unused var | Remove or prefix with `_` |
| splint | Nested assoc | Combine: `(assoc m :a 1 :b 2)` |
| splint | when-not pattern | Use `when-not` directly |

## Success Criteria

```bash
#!/bin/bash
# verify-quality.sh - Machine-readable success verification

verify_quality() {
  local exit_code=0
  local results=""

  # cljfmt
  if command -v clojure &>/dev/null && clojure -Ttools list 2>/dev/null | grep -q cljfmt; then
    if clojure -Tcljfmt check >/dev/null 2>&1; then
      results="${results}cljfmt:0;"
    else
      results="${results}cljfmt:1;"
      exit_code=1
    fi
  fi

  # clj-kondo
  if command -v clj-kondo &>/dev/null; then
    clj-kondo --lint src test >/dev/null 2>&1
    local kondo_exit=$?
    results="${results}clj-kondo:${kondo_exit};"
    [ $kondo_exit -eq 3 ] && exit_code=1  # Only fail on errors, not warnings
  fi

  # splint
  if command -v splint &>/dev/null; then
    if splint --check src test >/dev/null 2>&1; then
      results="${results}splint:0;"
    else
      results="${results}splint:1;"
      exit_code=1
    fi
  fi

  echo "results:${results}"
  echo "overall_exit:${exit_code}"
  return $exit_code
}

verify_quality
```

**Success = all of these true:**
- [ ] `cljfmt check` exits 0 (if installed)
- [ ] `clj-kondo --lint` exits 0 or 2 (0=clean, 2=warnings only, 3=errors)
- [ ] `splint --check` exits 0 (if installed)
- [ ] `/tmp/claude/quality-results.txt` shows `quality_check:pass`

## Related Skills

- `clj-kondo-linting` - Detailed clj-kondo config and all 100+ linters
- `cljfmt-formatting` - Detailed cljfmt indentation rules
- `splint-linting` - Detailed splint rules catalog
- `clojure-style` - Style conventions (prerequisite knowledge)
- `error-recovery` - When quality checks fail, use this to fix systematically
