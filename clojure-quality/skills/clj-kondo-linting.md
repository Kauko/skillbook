# clj-kondo Linting

Use when setting up or running clj-kondo for Clojure static analysis. clj-kondo is a powerful linter that catches errors, inconsistencies, and problematic patterns in Clojure code.

## Installation Check

Before proceeding, verify if clj-kondo is installed:

```bash
clj-kondo --version
```

If not installed, guide the user to install it:

**Via Homebrew (macOS/Linux):**
```bash
brew install borkdude/brew/clj-kondo
```

**Via Nix:**
```bash
nix-env -iA nixpkgs.clj-kondo
```

**Via Binary Download:**
Download from https://github.com/clj-kondo/clj-kondo/releases

**Verify installation:**
```bash
clj-kondo --version
```

## Configuration Setup

### 1. Create `.clj-kondo/config.edn`

Create a starter configuration with sensible defaults:

```clojure
{:linters {:unresolved-symbol {:level :error
                                :exclude [(clojure.core/defn)
                                          (cljs.core/defn)]}
           :unresolved-namespace {:level :error}
           :unresolved-var {:level :error}
           :invalid-arity {:level :error}
           :type-mismatch {:level :error}
           :missing-else-branch {:level :warning}
           :redundant-do {:level :warning}
           :redundant-let {:level :warning}
           :unused-binding {:level :warning
                            :exclude-destructured-as true}
           :unused-namespace {:level :warning
                              :exclude [clojure.test]}
           :unused-referred-var {:level :warning}
           :unused-private-var {:level :warning}
           :private-call {:level :error}
           :inline-def {:level :warning}
           :shadowed-var {:level :warning
                          :exclude [name]}
           :refer-all {:level :warning}
           :misplaced-docstring {:level :warning}
           :not-empty? {:level :warning}
           :deprecated-var {:level :warning}
           :unsorted-required-namespaces {:level :off}
           :refer {:level :off}
           :single-operand-comparison {:level :warning}
           :single-logical-operand {:level :warning}
           :format {:level :off}}
 :output {:pattern "{{filename}}:{{row}}:{{col}}: {{level}}: {{message}}"
          :canonical-paths true}
 :lint-as {}}
```

### 2. Common :lint-as Configurations

Many Clojure libraries define custom macros that clj-kondo needs help understanding. Add these to `:lint-as` as needed:

```clojure
:lint-as {malli.core/def schema.core/defschema
          malli.core/defn schema.core/defn
          clojure.test.check.properties/for-all clojure.core/let
          criterium.core/bench clojure.core/let
          criterium.core/quick-bench clojure.core/let
          compojure.core/defroutes clojure.core/def
          compojure.core/GET clojure.core/def
          compojure.core/POST clojure.core/def
          compojure.core/PUT clojure.core/def
          compojure.core/DELETE clojure.core/def
          compojure.core/context clojure.core/def
          integrant.core/defmethod-ig clojure.core/defmethod
          guardrails.core/defn clojure.core/defn
          guardrails.core/>defn clojure.core/defn
          guardrails.core/>defn- clojure.core/defn-
          clojure.core.async/go-loop clojure.core/loop}
```

### 3. Auto-Import Library Configurations

Many libraries ship with clj-kondo configuration. Import them automatically:

```bash
# Create cache and import configs from classpath
clj-kondo --lint "$(clojure -Spath)" --dependencies --parallel --copy-configs
```

This analyzes your dependencies (from `deps.edn`, `project.clj`, etc.) and copies library-specific configurations.

**Supported libraries include:**
- malli
- schema
- cljs.test
- re-frame
- fulcro
- integrant
- mount
- And many more...

Run this command whenever you add new dependencies.

## Linting Scripts

### Create `scripts/lint-kondo.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Running clj-kondo linter..."

if ! command -v clj-kondo &> /dev/null; then
    echo "Error: clj-kondo is not installed"
    echo "Install via: brew install borkdude/brew/clj-kondo"
    exit 1
fi

# Lint src and test directories
clj-kondo --lint src test "${@}"

echo "✓ clj-kondo linting completed"
```

Make it executable:
```bash
chmod +x scripts/lint-kondo.sh
```

**Usage:**
```bash
# Lint entire project
./scripts/lint-kondo.sh

# Lint with additional options
./scripts/lint-kondo.sh --parallel

# Lint specific file
clj-kondo --lint src/myapp/core.clj
```

## Git Pre-Commit Hook (Optional)

clj-kondo can run automatically before commits. The hook is **disabled by default**.

### Setup Instructions

Create `.git/hooks/pre-commit`:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Uncomment to enable clj-kondo pre-commit checks
# echo "Running clj-kondo..."
# if ! ./scripts/lint-kondo.sh; then
#     echo "clj-kondo found issues. Commit aborted."
#     echo "Fix issues or use 'git commit --no-verify' to skip."
#     exit 1
# fi

exit 0
```

Make it executable:
```bash
chmod +x .git/hooks/pre-commit
```

**To enable:** Uncomment the lines in the hook script.

**To bypass when enabled:** Use `git commit --no-verify`

## Understanding clj-kondo Reports

clj-kondo reports follow this format:

```
src/myapp/core.clj:42:5: warning: unused binding x
src/myapp/core.clj:45:1: error: unresolved symbol my-fn
```

**Report Structure:**
- **Location:** `file:line:column`
- **Level:** `error`, `warning`, or `info`
- **Message:** Description of the issue

### Common Issues and Fixes

**1. Unresolved Symbol**

```clojure
;; ERROR: unresolved symbol map-vals
(map-vals inc {:a 1})

;; FIX: Require the namespace
(require '[clojure.core :refer [map-vals]])
(map-vals inc {:a 1})
```

**Why:** The function doesn't exist in scope. Check spelling and imports.

**2. Invalid Arity**

```clojure
;; ERROR: invalid arity (expects 2, got 1)
(str/split "hello")

;; FIX: Provide required arguments
(str/split "hello" #" ")
```

**Why:** Function called with wrong number of arguments.

**3. Unused Binding**

```clojure
;; WARNING: unused binding result
(let [result (expensive-computation)
      x 42]
  x)

;; FIX: Remove unused binding or prefix with underscore
(let [_result (expensive-computation)
      x 42]
  x)
```

**Why:** Binding is never used. Prefix with `_` if intentionally ignored.

**4. Unused Namespace**

```clojure
;; WARNING: unused namespace clojure.set
(ns myapp.core
  (:require [clojure.set :as set]))

;; FIX: Remove unused require or use it
(ns myapp.core
  (:require [clojure.set :as set]))
(set/union #{1} #{2})
```

**Why:** Namespace is required but never used.

**5. Redundant Do**

```clojure
;; WARNING: redundant do
(if condition
  (do
    (println "yes")
    true)
  false)

;; FIX: Remove do (if has implicit do in branches)
(if condition
  (do
    (println "yes")
    true)
  false)
```

**Why:** `if` branches already have implicit `do`.

**6. Misplaced Docstring**

```clojure
;; WARNING: misplaced docstring
(defn my-fn
  [x]
  "Does something"
  (* x 2))

;; FIX: Place docstring before parameter vector
(defn my-fn
  "Does something"
  [x]
  (* x 2))
```

**Why:** Docstrings must come before parameter vectors.

**7. Private Call**

```clojure
;; ERROR: private call to other.ns/-private-fn
(other.ns/-private-fn)

;; FIX: Use public API or make function public
;; Either fix in other.ns or use public alternative
```

**Why:** Cannot call private functions from other namespaces.

**8. Not-Empty?**

```clojure
;; WARNING: use seq instead of not-empty?
(if (not (empty? coll))
  ...)

;; GOOD
(when (seq coll)
  ...)
```

**Why:** `(seq coll)` is the idiomatic nil-punning pattern in Clojure.

## Integration with clojure-mcp-light

If the project uses `clojure-mcp-light` MCP server, clj-kondo is automatically available through the MCP tool `lint_file`. This provides real-time linting during development.

**Check for clojure-mcp-light:**
```bash
# Look for MCP server configuration
cat ~/.config/claude/claude_desktop_config.json | grep -i clojure
```

If available, you can use:
- `lint_file` - Lint specific files via MCP
- Direct clj-kondo integration in editor

## CI Integration

Add to your CI pipeline:

**GitHub Actions:**
```yaml
# .github/workflows/lint.yml
- name: Install clj-kondo
  run: brew install borkdude/brew/clj-kondo

- name: Run clj-kondo
  run: ./scripts/lint-kondo.sh
```

**GitLab CI:**
```yaml
lint:
  stage: test
  script:
    - curl -sLO https://raw.githubusercontent.com/clj-kondo/clj-kondo/master/script/install-clj-kondo
    - chmod +x install-clj-kondo
    - ./install-clj-kondo
    - ./scripts/lint-kondo.sh
```

## Advanced Configuration

### Ignore Specific Warnings

**Inline:**
```clojure
;; Ignore next form
#_{:clj-kondo/ignore [:unused-binding]}
(let [x 42] nil)
```

**In config:**
```clojure
{:linters {:unused-binding {:level :off}}}
```

### Custom Hooks

For complex macros, write custom hooks:

```clojure
;; .clj-kondo/hooks/my_macro.clj
(ns hooks.my-macro
  (:require [clj-kondo.hooks-api :as api]))

(defn my-macro [{:keys [node]}]
  (let [[macro-name & args] (:children node)]
    {:node (api/list-node
            (list*
             (api/token-node 'do)
             args))}))
```

Register in config:
```clojure
{:hooks {:analyze-call {my.ns/my-macro hooks.my-macro/my-macro}}}
```

## Best Practices

1. **Import library configs:** Run `--copy-configs` after adding dependencies
2. **Keep config minimal:** Start with defaults, adjust as needed
3. **Address errors first:** Focus on `:error` level before `:warning`
4. **Use inline ignores sparingly:** Fix issues rather than ignoring them
5. **Run in editor:** Configure editor integration for real-time feedback
6. **CI enforcement:** Fail builds on clj-kondo errors

## Troubleshooting

**False positives for custom macros:**
- Add `:lint-as` configuration
- Write custom hook if needed

**Performance issues:**
- Use `--parallel` flag
- Exclude large generated files via `:output {:exclude-files [...]}`

**Cache issues:**
- Delete `.clj-kondo/.cache` and re-run

## Resources

- **clj-kondo GitHub:** https://github.com/clj-kondo/clj-kondo
- **Configuration:** https://github.com/clj-kondo/clj-kondo/blob/master/doc/config.md
- **Linters:** https://github.com/clj-kondo/clj-kondo/blob/master/doc/linters.md
- **Hooks:** https://github.com/clj-kondo/clj-kondo/blob/master/doc/hooks.md
- **Editor Integration:** https://github.com/clj-kondo/clj-kondo/blob/master/doc/editor-integration.md
