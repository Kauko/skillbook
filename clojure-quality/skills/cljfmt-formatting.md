# cljfmt Formatting

Use when setting up or running cljfmt for Clojure code formatting. cljfmt automatically formats Clojure code according to community standards and customizable rules.

## Quick Reference

For detailed documentation:
- **Configuration options:** See `references/configuration.md` for all .cljfmt.edn options
- **Indentation rules:** See `references/indentation.md` for custom indentation patterns
- **Official repository:** https://github.com/weavejester/cljfmt

## Installation Check

Before proceeding, verify if cljfmt is installed:

```bash
clojure -Ttools list
```

Look for `cljfmt` in the tool list. If not present, install it:

**Install as Clojure CLI tool (recommended):**
```bash
clojure -Ttools install io.github.weavejester/cljfmt '{:git/tag "0.15.6"}' :as cljfmt
```

**Verify installation:**
```bash
clojure -Tcljfmt help
```

**Alternative: Standalone binary**
For Linux/macOS:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/weavejester/cljfmt/HEAD/install.sh)"
```

**Alternative: Leiningen plugin**
Add to `project.clj`:
```clojure
:plugins [[dev.weavejester/lein-cljfmt "0.15.6"]]
```

## Configuration Setup

### 1. Create `.cljfmt.edn`

Create a configuration file aligned with Clojure Style Guide:

```clojure
{:indents {;; Core macros
           clojure.core/defprotocol [[:inner 0]]
           clojure.core/extend-protocol [[:block 1]]
           clojure.core/extend-type [[:block 1]]
           clojure.core/reify [[:inner 0]]
           clojure.core/proxy [[:inner 0] [:inner 1]]
           clojure.core/deftype [[:block 2] [:inner 2]]
           clojure.core/defrecord [[:block 2] [:inner 2]]

           ;; Testing
           clojure.test/deftest [[:inner 0]]
           clojure.test/testing [[:inner 0]]
           clojure.test/use-fixtures [[:inner 0]]

           ;; Spec
           clojure.spec.alpha/fdef [[:block 1]]

           ;; Schema
           schema.core/defschema [[:inner 0]]
           schema.core/defn [[:inner 0]]
           schema.core/defmethod [[:inner 0]]

           ;; Malli
           malli.core/def [[:inner 0]]
           malli.core/defn [[:inner 0]]

           ;; Compojure
           compojure.core/defroutes [[:block 0]]
           compojure.core/GET [[:block 2]]
           compojure.core/POST [[:block 2]]
           compojure.core/PUT [[:block 2]]
           compojure.core/DELETE [[:block 2]]
           compojure.core/PATCH [[:block 2]]
           compojure.core/context [[:block 2]]

           ;; Ring
           ring.util.response/response [[:block 0]]

           ;; Core.async
           clojure.core.async/go [[:inner 0]]
           clojure.core.async/go-loop [[:block 1]]
           clojure.core.async/thread [[:inner 0]]

           ;; Re-frame
           re-frame.core/reg-event-db [[:inner 0]]
           re-frame.core/reg-event-fx [[:inner 0]]
           re-frame.core/reg-sub [[:inner 0]]
           re-frame.core/reg-fx [[:inner 0]]
           re-frame.core/reg-cofx [[:inner 0]]

           ;; Integrant
           integrant.core/defmethod-ig [[:inner 0]]

           ;; Component
           component/start [[:inner 0]]
           component/stop [[:inner 0]]

           ;; Mount
           mount.core/defstate [[:inner 0]]

           ;; Guardrails
           guardrails.core/defn [[:inner 0]]
           guardrails.core/>defn [[:inner 0]]
           guardrails.core/>defn- [[:inner 0]]

           ;; Common patterns
           #".*-for-all$" [[:block 1]]
           #"^def.*" [[:inner 0]]
           #"^with-.*" [[:inner 1]]
           #"^let.*" [[:block 1]]}

 :indentation? true
 :remove-surrounding-whitespace? true
 :remove-trailing-whitespace? true
 :insert-missing-whitespace? true
 :remove-consecutive-blank-lines? true
 :max-consecutive-blank-lines 1
 :indents? true
 :alias-map {}
 :paths ["src" "test" "dev"]}
```

**Key Configuration Options:**

- `:extra-indents` - Add custom indentation rules (recommended, preserves defaults)
- `:indents` - Replace ALL default indentation rules (use with caution)
- `:indentation?` - Enable smart indentation (default: true)
- `:remove-surrounding-whitespace?` - Clean up extra spaces (default: true)
- `:remove-trailing-whitespace?` - Remove line-ending spaces (default: true)
- `:insert-missing-whitespace?` - Add required spaces (default: true)
- `:remove-consecutive-blank-lines?` - Limit blank lines (default: true)
- `:max-consecutive-blank-lines` - Maximum blank lines allowed (default: 2)
- `:paths` - Directories to format (default: ["src" "test"])
- `:parallel?` - Process files concurrently for better performance
- `:function-arguments-indentation` - Choose style: `:community`, `:cursive`, or `:zprint`

See `references/configuration.md` for complete list of all options.

### 2. Understanding Indent Rules

cljfmt uses indent patterns to format macro calls correctly. Three main rule types:

**`:inner` - Consistent two-space indentation:**
```clojure
{:extra-indents {defn [[:inner 0]]}}

(defn my-function
  [args]
  body)
```

**`:block` - Block-style indentation:**
```clojure
{:extra-indents {when [[:block 1]]}}

(when condition
  first-expr
  second-expr)
```

**Regex patterns** - Match multiple macros:
```clojure
{:extra-indents {#"^def.*" [[:inner 0]]    ; defn, defmacro, etc.
                 #"^with-.*" [[:inner 1]]}} ; with-open, with-redefs, etc.
```

See `references/indentation.md` for comprehensive guide to all indentation rule types, pattern matching, and advanced techniques.

## Formatting Scripts

### Create `scripts/fmt-check.sh`

Check formatting without modifying files:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Checking code formatting with cljfmt..."

if ! clojure -Tcljfmt help &> /dev/null; then
    echo "Error: cljfmt is not installed"
    echo "Install via: clojure -Ttools install io.github.weavejester/cljfmt '{:git/tag \"0.15.6\"}' :as cljfmt"
    exit 1
fi

# Check formatting (exits with error if files need formatting)
clojure -Tcljfmt check

echo "✓ All files are properly formatted"
```

### Create `scripts/fmt-fix.sh`

Format files in place:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Formatting code with cljfmt..."

if ! clojure -Tcljfmt help &> /dev/null; then
    echo "Error: cljfmt is not installed"
    echo "Install via: clojure -Ttools install io.github.weavejester/cljfmt '{:git/tag \"0.15.6\"}' :as cljfmt"
    exit 1
fi

# Format all files in place
clojure -Tcljfmt fix

echo "✓ Code formatting completed"
```

Make scripts executable:
```bash
chmod +x scripts/fmt-check.sh scripts/fmt-fix.sh
```

**Usage:**
```bash
# Check formatting (CI/pre-commit)
./scripts/fmt-check.sh

# Fix formatting
./scripts/fmt-fix.sh

# Format specific files
clojure -Tcljfmt fix :paths '["src/myapp/core.clj"]'
```

## Git Pre-Commit Hook (Optional)

cljfmt can check formatting before commits. The hook is **disabled by default** and runs in **check-only mode**.

### Setup Instructions

Create `.git/hooks/pre-commit`:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Uncomment to enable cljfmt pre-commit checks
# echo "Checking code formatting..."
# if ! ./scripts/fmt-check.sh; then
#     echo "Code formatting issues found. Commit aborted."
#     echo "Run './scripts/fmt-fix.sh' to fix formatting."
#     echo "Or use 'git commit --no-verify' to skip."
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

**Note:** The hook runs in **check mode** only. It does not automatically format files. This ensures you review changes before committing.

## Understanding cljfmt Output

When checking format, cljfmt shows files that need formatting:

```
Checking: src
src/myapp/core.clj
src/myapp/utils.clj
2 file(s) require formatting
```

When fixing, it shows files modified:

```
Reformatting: src
Formatted src/myapp/core.clj
Formatted src/myapp/utils.clj
2 file(s) formatted
```

## Common Formatting Patterns

### Indentation

**Function definitions:**
```clojure
;; GOOD
(defn my-function
  "Docstring"
  [arg1 arg2]
  (+ arg1 arg2))

;; BAD (cljfmt will fix)
(defn my-function
"Docstring"
[arg1 arg2]
(+ arg1 arg2))
```

**Let bindings:**
```clojure
;; GOOD
(let [x 1
      y 2
      z (+ x y)]
  (* z 2))

;; BAD (cljfmt will fix)
(let [x 1
y 2
z (+ x y)]
(* z 2))
```

**Threading macros:**
```clojure
;; GOOD
(-> data
    (assoc :key value)
    (update :other inc)
    (dissoc :old))

;; BAD (cljfmt will fix)
(-> data
  (assoc :key value)
  (update :other inc)
  (dissoc :old))
```

### Whitespace

**Surrounding forms:**
```clojure
;; GOOD
(foo bar baz)

;; BAD (cljfmt will fix)
( foo bar baz )
```

**Trailing whitespace:**
```clojure
;; GOOD
(defn foo []
  :bar)

;; BAD (cljfmt will fix - invisible trailing spaces)
(defn foo []
  :bar)
```

**Consecutive blank lines:**
```clojure
;; GOOD (single blank line)
(defn foo [] :a)

(defn bar [] :b)

;; BAD (multiple blank lines - cljfmt will fix)
(defn foo [] :a)


(defn bar [] :b)
```

## CI Integration

Add to your CI pipeline:

**GitHub Actions:**
```yaml
# .github/workflows/format.yml
- name: Install cljfmt
  run: clojure -Ttools install io.github.weavejester/cljfmt '{:git/tag "0.15.6"}' :as cljfmt

- name: Check formatting
  run: ./scripts/fmt-check.sh
```

**GitLab CI:**
```yaml
format:
  stage: test
  script:
    - clojure -Ttools install io.github.weavejester/cljfmt '{:git/tag "0.15.6"}' :as cljfmt
    - ./scripts/fmt-check.sh
```

## Editor Integration

### Emacs (CIDER)

Add to `.emacs.d/init.el`:
```elisp
;; Format buffer on save
(add-hook 'clojure-mode-hook
          (lambda ()
            (add-hook 'before-save-hook
                      (lambda ()
                        (when (eq major-mode 'clojure-mode)
                          (shell-command-on-region
                           (point-min) (point-max)
                           "clojure -Tcljfmt fix"
                           (current-buffer) t))))))
```

### VS Code (Calva)

Calva has built-in formatting. Configure in `settings.json`:
```json
{
  "calva.fmt.configPath": ".cljfmt.edn",
  "editor.formatOnSave": true
}
```

### IntelliJ (Cursive)

Cursive uses its own formatter. To use cljfmt:
1. Install "File Watchers" plugin
2. Configure file watcher to run `clojure -Tcljfmt fix` on save

## Advanced Configuration

### Project-Specific Indents

Add custom indents for project macros using `:extra-indents`:

```clojure
{:extra-indents {myapp.macros/with-context [[:inner 1]]
                 myapp.routes/defapi [[:block 0]]
                 #"^def.*" [[:inner 0]]}}
```

Note: Use `:extra-indents` to preserve default rules. Using `:indents` replaces ALL defaults.

### Namespace Aliases

Specify namespace aliases for proper indentation:

```clojure
{:alias-map {test clojure.test
             async clojure.core.async
             s schema.core}}
```

### Performance and Paths

Configure paths and enable parallel processing:

```clojure
{:paths ["src" "test" "dev"]
 :exclude-paths ["src/generated"]
 :parallel? true}
```

### File Extensions

Format non-standard extensions:

```clojure
{:file-pattern #"\.clj[scx]?$"}
```

For complete configuration reference, see `references/configuration.md` and `references/indentation.md`.

## Best Practices

1. **Format early and often:** Run `./scripts/fmt-fix.sh` regularly
2. **CI enforcement:** Use `fmt-check.sh` in CI to enforce consistency
3. **Team alignment:** Commit `.cljfmt.edn` to version control
4. **Editor integration:** Configure your editor to format on save
5. **Pre-commit checks:** Consider enabling the pre-commit hook for teams
6. **Custom indents:** Document project-specific indentation rules in `.cljfmt.edn`

## Troubleshooting

**Incorrect indentation for custom macros:**
- Add indent rules to `:extra-indents` in `.cljfmt.edn` (not `:indents`)
- Use regex patterns for multiple similar macros: `#"^with-.*" [[:inner 1]]`
- Check namespace qualification and `:alias-map` configuration
- See `references/indentation.md` for detailed guidance

**Lost default indentation rules:**
- You likely used `:indents` instead of `:extra-indents`
- Change to `:extra-indents` to preserve defaults
- Or add `:legacy/merge-indents? true` for backward compatibility

**Formatting conflicts with other tools:**
- Ensure all team members use same `.cljfmt.edn`
- Disable conflicting editor formatters
- Commit `.cljfmt.edn` to version control

**Performance issues on large projects:**
- Enable `:parallel? true` for concurrent processing
- Limit `:paths` to necessary directories
- Use `:exclude-paths` for generated code

**Tool not found:**
- Verify installation: `clojure -Ttools list`
- Re-install: `clojure -Ttools install io.github.weavejester/cljfmt '{:git/tag "0.15.6"}' :as cljfmt`

## Resources

### Local Documentation
- **Configuration Reference:** `references/configuration.md` - Complete guide to all .cljfmt.edn options
- **Indentation Reference:** `references/indentation.md` - Comprehensive guide to custom indentation rules

### External Resources
- **cljfmt GitHub:** https://github.com/weavejester/cljfmt
- **Clojure Style Guide:** https://guide.clojure.style
- **Clojure CLI Tools:** https://clojure.org/reference/deps_and_cli
