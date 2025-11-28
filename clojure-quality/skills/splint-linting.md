---
name: splint-linting
description: Use when user wants to improve code idioms, find non-idiomatic patterns, or get style suggestions beyond what clj-kondo provides.
requires:
  tools: [splint]
  skills: []
---

# Splint Linting

Splint analyzes code shape and idioms without executing it. Complements clj-kondo (which does semantic analysis).

## Prerequisites

```bash
command -v splint >/dev/null || { echo "Install: brew install noahtheduke/tap/splint"; exit 1; }
```

## Usage

```bash
# Lint project
splint src test

# With autocorrect
splint --fix src

# Check only (CI)
splint --check src test
```

## Configuration

`.splint.edn`:
```clojure
{:rules
 {:lint/if-let-else-nil {:enabled true}
  :lint/assoc-assoc {:enabled true}
  :style/when-not-call {:enabled true}
  :performance/get-keyword {:enabled true}}

 :excludes ["dev/" "resources/"]}
```

## Key Rules

| Rule | Transforms |
|------|------------|
| `lint/assoc-assoc` | `(assoc (assoc m :a 1) :b 2)` → `(assoc m :a 1 :b 2)` |
| `lint/if-let-else-nil` | `(if-let [x y] x nil)` → `(when-let [x y] x)` |
| `style/when-not-call` | `(when (not x) ...)` → `(when-not x ...)` |
| `style/prefer-condp` | Repeated `=` in cond → `condp` |
| `performance/get-keyword` | `(get m :k)` → `(:k m)` |
| `naming/single-segment-ns` | Warns on single-segment namespaces |

## CI Integration

```yaml
- name: Check idioms
  run: splint --check src test
```

## Difference from clj-kondo

| Tool | Focus |
|------|-------|
| clj-kondo | Semantic (arity, unused vars, types) |
| Splint | Syntactic (idioms, style, code shape) |

Use both for comprehensive linting.

## Reference Documentation

- `references/configuration.md` - Config options
- `references/rules.md` - All rules catalog
- `references/usage.md` - CLI options

## Success Criteria

- [ ] `splint --check src test` passes or shows only intentional exceptions
- [ ] Non-idiomatic patterns addressed or documented as exceptions

## Related Skills

- `clj-kondo-linting` - Semantic analysis
- `cljfmt-formatting` - Code formatting
- `clojure-style` - Style conventions
