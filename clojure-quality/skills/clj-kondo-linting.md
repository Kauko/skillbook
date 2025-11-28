---
name: clj-kondo-linting
description: Use when user wants to find bugs in Clojure code, check for unused vars, detect arity errors, or set up static analysis.
requires:
  tools: [clj-kondo]
  skills: []
---

# clj-kondo Linting

## Prerequisites

```bash
command -v clj-kondo >/dev/null || { echo "Install: brew install borkdude/brew/clj-kondo"; exit 1; }
```

Initialize for project (analyzes dependencies):
```bash
clj-kondo --lint "$(clojure -Spath)" --dependencies --parallel --copy-configs
```

## Configuration

`.clj-kondo/config.edn`:
```clojure
{:linters
 {:unused-namespace {:level :warning}
  :unused-binding {:level :warning}
  :unresolved-symbol {:level :error}}

 :output
 {:format :text
  :include-files ["src/" "test/"]}

 ;; Library-specific configs (auto-copied by --copy-configs)
 :config-paths ["resources/clj-kondo.exports"]}
```

## Usage

```bash
# Lint project
clj-kondo --lint src test

# Lint with specific format
clj-kondo --lint src --config '{:output {:format :edn}}'

# Watch mode (with entr)
find src -name "*.clj" | entr clj-kondo --lint src
```

## Key Linters

| Linter | Catches |
|--------|---------|
| `:unresolved-symbol` | Missing vars/requires |
| `:unused-binding` | Unused let bindings |
| `:unused-namespace` | Unused requires |
| `:type-mismatch` | Wrong arg types |
| `:invalid-arity` | Wrong arg count |
| `:redundant-do` | Unnecessary do blocks |
| `:deprecated-var` | Using deprecated functions |

## Editor Integration

**Emacs (flycheck-clj-kondo):**
```elisp
(require 'flycheck-clj-kondo)
```

**VSCode (Calva):** Built-in, works automatically.

**IntelliJ (Cursive):** Configure as external tool.

## CI Integration

```yaml
# GitHub Actions
- name: Lint
  run: clj-kondo --lint src test
```

## Reference Documentation

- `references/configuration.md` - Complete config options
- `references/linters.md` - All 100+ linters
- `references/hooks.md` - Custom macro analysis

## Success Criteria

- [ ] `.clj-kondo/config.edn` exists with project settings
- [ ] `clj-kondo --lint src test` runs without errors
- [ ] No unresolved symbols or arity mismatches

## Related Skills

- `clojure-style` - Style conventions
- `cljfmt-formatting` - Code formatting
- `splint-linting` - Idiom linting
