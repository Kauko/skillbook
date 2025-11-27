# Splint Linting

Use when setting up or running Splint for Clojure idiom linting. Splint is a linting tool that focuses on Clojure idioms and style, catching patterns that are technically valid but non-idiomatic.

## Installation Check

Before proceeding, verify if Splint is installed:

```bash
splint --version
```

If not installed, guide the user to install it:

**Via Homebrew (macOS/Linux):**
```bash
brew install noahtheduke/tap/splint
```

**Via Binary Download:**
Download the latest release from https://github.com/NoahTheDuke/splint/releases

**Verify installation:**
```bash
splint --version
```

## Configuration Setup

### 1. Create `.splint.edn` in Project Root

Create a `.splint.edn` configuration file with sensible defaults:

```clojure
{:paths ["src" "test"]
 :output {:pattern "full"}
 :rules {:style/prefer-clj-string {:enabled true}
         :style/prefer-clj-math {:enabled true}
         :style/prefer-condp {:enabled true}
         :style/prefer-vary-meta {:enabled true}
         :style/single-key-in {:enabled true}
         :style/assoc-assoc {:enabled true}
         :style/nested-addition {:enabled true}
         :style/nested-multiplication {:enabled true}
         :style/prefer-while {:enabled true}
         :style/useless-do {:enabled true}
         :style/redundant-call {:enabled true}
         :naming/lisp-case {:enabled true}
         :naming/single-segment-namespace {:enabled true}
         :performance/dot-class-method {:enabled true}
         :performance/get-in-literals {:enabled true}
         :performance/assoc-many {:enabled true}
         :lint/if-let-else {:enabled true}
         :lint/if-not-both {:enabled true}
         :lint/if-same-truthy {:enabled true}
         :lint/if-nil-else {:enabled true}
         :lint/redundant-let {:enabled true}
         :lint/redundant-do {:enabled true}
         :lint/body-unquote-splicing {:enabled true}
         :lint/duplicate-field-name {:enabled true}}}
```

**Key Configuration Options:**

- `:paths` - Directories to lint (typically `["src" "test"]`)
- `:output` - Output format: `"simple"`, `"full"`, `"json"`
- `:rules` - Enable/disable specific linting rules

### 2. Create Linting Script

Create `scripts/lint-splint.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Running Splint linter..."

if ! command -v splint &> /dev/null; then
    echo "Error: Splint is not installed"
    echo "Install via: brew install noahtheduke/tap/splint"
    exit 1
fi

splint "${@:-.}"

echo "✓ Splint linting completed"
```

Make it executable:
```bash
chmod +x scripts/lint-splint.sh
```

**Usage:**
```bash
# Lint entire project
./scripts/lint-splint.sh

# Lint specific path
./scripts/lint-splint.sh src/myapp/core.clj

# Check if any issues exist (for CI)
./scripts/lint-splint.sh || exit 1
```

## Git Pre-Commit Hook (Optional)

Splint can run automatically before commits. The hook is **disabled by default** to avoid disrupting workflows.

### Setup Instructions

Create `.git/hooks/pre-commit`:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Uncomment to enable Splint pre-commit checks
# echo "Running Splint..."
# if ! ./scripts/lint-splint.sh; then
#     echo "Splint found issues. Commit aborted."
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

## Understanding Splint Reports

Splint reports follow this format:

```
path/to/file.clj:42:5 [style/prefer-condp] Use condp instead of multiple cond predicates
  (cond
    (= x 1) :one
    (= x 2) :two)
```

**Report Structure:**
- **Location:** `file.clj:line:column`
- **Rule:** `[category/rule-name]`
- **Message:** Human-readable explanation
- **Code Context:** The problematic code

### Common Issues and Fixes

**1. `style/prefer-clj-string` - Use clojure.string functions**

```clojure
;; BAD
(.toLowerCase s)

;; GOOD
(require '[clojure.string :as str])
(str/lower-case s)
```

**Why:** Clojure.string functions are more idiomatic and portable.

**2. `style/prefer-condp` - Use condp for repeated predicates**

```clojure
;; BAD
(cond
  (= x 1) :one
  (= x 2) :two
  (= x 3) :three)

;; GOOD
(condp = x
  1 :one
  2 :two
  3 :three)
```

**Why:** `condp` eliminates repetition and makes the pattern explicit.

**3. `style/assoc-assoc` - Merge multiple assocs**

```clojure
;; BAD
(assoc (assoc m :a 1) :b 2)

;; GOOD
(assoc m :a 1 :b 2)
```

**Why:** Single `assoc` is more efficient and readable.

**4. `naming/lisp-case` - Use kebab-case naming**

```clojure
;; BAD
(defn myFunction [myArg]
  ...)

;; GOOD
(defn my-function [my-arg]
  ...)
```

**Why:** Kebab-case is the Clojure community standard.

**5. `lint/redundant-let` - Remove unnecessary let bindings**

```clojure
;; BAD
(let [x 42]
  x)

;; GOOD
42
```

**Why:** Direct values are clearer than unnecessary bindings.

**6. `performance/get-in-literals` - Use direct access for literal paths**

```clojure
;; BAD
(get-in m [:a :b])

;; GOOD
(-> m :a :b)
```

**Why:** Threading is more performant and idiomatic for known paths.

## CI Integration

Add to GitHub Actions, GitLab CI, or other CI systems:

```yaml
# .github/workflows/lint.yml
- name: Run Splint
  run: |
    brew install noahtheduke/tap/splint
    ./scripts/lint-splint.sh
```

## Best Practices

1. **Run regularly:** Make `./scripts/lint-splint.sh` part of your development workflow
2. **Fix incrementally:** Address new issues as they arise rather than fixing all at once
3. **Team alignment:** Discuss and agree on which rules to enable/disable
4. **CI enforcement:** Run Splint in CI to prevent non-idiomatic code from merging
5. **Document exceptions:** If disabling a rule, document why in `.splint.edn`

## Disabling Rules

To disable specific rules, set `:enabled false`:

```clojure
{:rules {:style/prefer-condp {:enabled false}}}
```

To disable a rule for specific code, use inline comments:

```clojure
; splint/disable style/prefer-condp
(cond
  (= x 1) :one
  (= x 2) :two)
; splint/enable style/prefer-condp
```

## Resources

- **Splint GitHub:** https://github.com/NoahTheDuke/splint
- **Rule Documentation:** https://github.com/NoahTheDuke/splint/blob/main/docs/rules.md
- **Configuration Guide:** https://github.com/NoahTheDuke/splint/blob/main/docs/configuration.md
