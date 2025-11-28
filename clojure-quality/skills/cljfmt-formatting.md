---
name: cljfmt-formatting
description: Use when user wants to format Clojure code, fix indentation, or enforce consistent code style across a project.
requires:
  tools: [clojure]
  skills: []
---

# cljfmt Formatting

## Prerequisites

```bash
command -v clojure >/dev/null || { echo "Install: brew install clojure/tools/clojure"; exit 1; }
clojure -Ttools list 2>/dev/null | grep -q cljfmt || echo "Install: clojure -Ttools install io.github.weavejester/cljfmt '{:git/tag \"0.15.6\"}' :as cljfmt"
```

## Configuration

`.cljfmt.edn`:
```clojure
{:indentation? true
 :remove-surrounding-whitespace? true
 :remove-trailing-whitespace? true
 :remove-consecutive-blank-lines? true
 :insert-missing-whitespace? true
 :sort-ns-references? true

 ;; Custom indentation rules
 :indents {defrecord [[:inner 0]]}

 ;; Paths to format
 :paths ["src" "test"]}
```

## Usage

```bash
# Check formatting (CI)
clojure -Tcljfmt check

# Fix formatting
clojure -Tcljfmt fix

# Format specific files
clojure -Tcljfmt fix --file src/myapp/core.clj
```

## Key Options

| Option | Default | Effect |
|--------|---------|--------|
| `:indentation?` | true | Apply indentation rules |
| `:sort-ns-references?` | false | Sort require/import |
| `:remove-consecutive-blank-lines?` | true | Max 1 blank line |
| `:split-keypaths?` | false | One key per line in maps |

## Indentation Rules

```clojure
;; Custom form indentation
{:indents
 {defrecord [[:inner 0]]       ; Body at base indent
  with-db [[:block 1]]         ; First arg block, rest body
  defroutes [[:inner 0]]}}     ; All args as body
```

## Editor Integration

**Emacs:** `cljfmt-mode`
**VSCode (Calva):** Built-in, use "Format Document"
**IntelliJ (Cursive):** Configure in preferences

## CI Integration

```yaml
- name: Check formatting
  run: clojure -Tcljfmt check || (echo "Run 'clojure -Tcljfmt fix'" && exit 1)
```

## Reference Documentation

- `references/configuration.md` - All config options
- `references/indentation.md` - Indentation rule syntax

## Success Criteria

- [ ] `.cljfmt.edn` configuration exists
- [ ] `clojure -Tcljfmt check` passes with no violations
- [ ] Code follows consistent indentation

## Related Skills

- `clojure-style` - Style conventions
- `clj-kondo-linting` - Static analysis
