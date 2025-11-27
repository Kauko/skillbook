# Splint Linting

Use when setting up or running Splint for Clojure idiom linting. Splint is a fast linting tool focused on Clojure style and code shape, catching patterns that are technically valid but non-idiomatic.

## Overview

**What Splint Does:**
- Analyzes code structure without executing it
- Warns about guidelines from the Clojure Style Guide
- Focuses on code shape and idioms, not semantic meaning
- Provides autocorrect capabilities for safe transformations

**What Splint Does NOT Do:**
- Execute code or expand macros
- Perform lexical analysis (use clj-kondo for that)
- Track variable usage or binding correctness

**Performance**: Splint is fast - processing 223 files in ~6 seconds vs. Kibit's 34+ minutes on the same codebase.

## Installation Check

Before proceeding, verify if Splint is installed:

```bash
splint --version
```

If not installed, guide the user to install it:

### Via Homebrew (macOS/Linux)

```bash
brew install noahtheduke/tap/splint
```

### Via Clojure CLI

Add to `deps.edn`:

```clojure
:aliases {:splint {:extra-deps {io.github.noahtheduke/splint {:mvn/version "1.22.0"}}
                   :main-opts ["-m" "noahtheduke.splint"]}}
```

Run with: `clojure -M:splint [args...]`

### Via Leiningen

Add to `project.clj`:

```clojure
:profiles {:dev {:dependencies [[io.github.noahtheduke/splint "1.22.0"]]}}
:aliases {"splint" ["run" "-m" "noahtheduke.splint"]}
```

Run with: `lein splint [args...]`

### Via Babashka

Requires version 1.12.205+. Add to `bb.edn`:

```clojure
:tasks {splint {:extra-deps {io.github.noahtheduke/splint {:mvn/version "1.22.0"}}
                :task noahtheduke.splint/-main}}
```

Run with: `bb splint [args...]`

### Verify Installation

```bash
splint --version
```

## Quick Start Workflow

### For New Projects

1. **Create configuration file** at project root:

```bash
cat > .splint.edn << 'EOF'
{:paths ["src" "test"]
 :output {:pattern "full"}
 :rules {:lint {:enabled true}
         :style {:enabled true}
         :naming {:enabled true}
         :performance {:enabled false}
         :metrics {:enabled false}}}
EOF
```

2. **Run Splint**:

```bash
splint
```

3. **Fix issues** as you develop.

### For Existing Projects

1. **Generate baseline configuration**:

```bash
splint --auto-gen-config
```

This creates `.splint.edn` with all currently failing rules disabled.

2. **Review the configuration**:

```bash
cat .splint.edn
```

3. **Enable rules incrementally**:

Edit `.splint.edn` to enable one rule or category at a time:

```clojure
{:rules {:style/eq-true {:enabled true}}}
```

4. **Run Splint and fix issues**:

```bash
splint
```

5. **Repeat**: Enable another rule, fix issues, commit.

## Configuration Setup

### 1. Basic Configuration File

Create `.splint.edn` in project root:

```clojure
{:paths ["src" "test"]
 :output {:pattern "full"}
 :rules {:lint {:enabled true}
         :style {:enabled true}
         :naming {:enabled true}
         :performance {:enabled false}
         :metrics {:enabled false}}
 :global {:excludes ["glob:**/generated/**"
                     "glob:**/target/**"]}}
```

**Key Configuration Options:**

- `:paths` - Directories to lint (typically `["src" "test"]`)
- `:output` - Output format: `"simple"`, `"full"`, `"json"`, `"markdown"`, `"clj-kondo"`
- `:rules` - Enable/disable specific linting rules or entire categories
- `:global :excludes` - File patterns to exclude from linting

**For detailed configuration options, see**: `references/configuration.md`

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

## Understanding Splint Reports

Splint reports follow this format:

```
path/to/file.clj:42:5 [style/prefer-condp] Use condp instead of cond
  (cond
    (= x 1) :one
    (= x 2) :two)
=>
  (condp = x
    1 :one
    2 :two)
```

**Report Structure:**
- **Location**: `file.clj:line:column`
- **Rule**: `[category/rule-name]`
- **Message**: Human-readable explanation
- **Code Context**: The problematic code
- **Suggestion**: Recommended replacement (with `=>`)

## Rule Categories

Splint organizes rules into five main categories:

### Lint Rules (Enabled by Default)

Catch potentially incorrect or suspicious patterns:
- Control flow issues (missing branches, redundant conditions)
- Function and data transformation problems
- Java interop concerns
- Code quality and safety issues

**Examples**: `lint/redundant-let`, `lint/if-else-nil`, `lint/duplicate-case-test`

### Style Rules (Enabled by Default)

Enforce Clojure community idioms:
- String operations (`prefer-clj-string`)
- Comparison functions (`eq-true`, `pos-checks`)
- Collection operations (`filter-vec-filterv`)
- Control flow (`prefer-condp`, `when-not-call`)

**Examples**: `style/prefer-clj-string`, `style/prefer-condp`, `style/eq-true`

### Naming Rules (Enabled by Default)

Enforce naming conventions:
- `lisp-case` - Use kebab-case for functions and variables
- `conventional-aliases` - Use standard library aliases (`:as str` not `:as string`)
- `predicate` - Boolean functions should end with `?`
- `record-name` - Records use PascalCase
- `single-segment-namespace` - Require multi-segment namespaces

### Performance Rules (Disabled by Default)

Suggest faster alternatives (more contentious):
- `assoc-many` - Chain `assoc` calls instead of multiple key-value pairs
- `get-in-literals` - Use threading instead of `get-in` for literal paths
- `get-keyword` - Use keywords as functions instead of `get`
- `into-transducer` - Use 4-arity `into` for transducers
- `single-literal-merge` - Use `assoc` instead of `merge` for literals

**Note**: These are disabled by default. Enable selectively after team discussion.

### Metrics Rules (Disabled by Default)

Measure code properties:
- `fn-length` - Enforce maximum function length (default: 10 lines)
- `parameter-count` - Limit function parameters (default: 4)

**Note**: Configure thresholds based on team standards.

**For complete rule documentation, see**: `references/rules.md`

## Common Fixes

### 1. Use Clojure.string Functions

```clojure
; Bad - style/prefer-clj-string
(.toLowerCase s)

; Good
(require '[clojure.string :as str])
(str/lower-case s)
```

**Why**: Clojure.string functions are more idiomatic and portable.

### 2. Use condp for Repeated Predicates

```clojure
; Bad - style/prefer-condp
(cond
  (= x 1) :one
  (= x 2) :two
  (= x 3) :three)

; Good
(condp = x
  1 :one
  2 :two
  3 :three)
```

**Why**: `condp` eliminates repetition and makes the pattern explicit.

### 3. Merge Multiple assocs

```clojure
; Bad - style/assoc-assoc
(assoc (assoc m :a 1) :b 2)

; Good
(assoc m :a 1 :b 2)
```

**Why**: Single `assoc` is more efficient and readable.

### 4. Use Kebab-case Naming

```clojure
; Bad - naming/lisp-case
(defn myFunction [myArg]
  ...)

; Good
(defn my-function [my-arg]
  ...)
```

**Why**: Kebab-case is the Clojure community standard.

### 5. Remove Unnecessary Let Bindings

```clojure
; Bad - lint/redundant-let
(let [x 42]
  x)

; Good
42
```

**Why**: Direct values are clearer than unnecessary bindings.

### 6. Use when Instead of if Without Else

```clojure
; Bad - lint/if-else-nil
(if condition :value nil)

; Good
(when condition :value)
```

**Why**: `when` is more idiomatic for single-branch conditionals.

### 7. Use Predicate Functions

```clojure
; Bad - style/eq-true
(= true x)

; Good
(true? x)
```

**Why**: Predicate functions are clearer and more idiomatic.

**For more examples and fixes, see**: `references/rules.md`

## Autocorrect

Splint can automatically fix many issues:

```bash
splint --autocorrect
```

**WARNING**: Autocorrect removes all comments and uneval blocks. Always commit your code before running autocorrect.

**Recommended workflow**:
1. Commit your changes: `git add . && git commit -m "Before autocorrect"`
2. Run autocorrect: `splint --autocorrect`
3. Review changes: `git diff`
4. Test your code: `lein test` or equivalent
5. Commit fixes: `git add . && git commit -m "Apply Splint autocorrect"`

**Autocorrect specific rules only**:
```bash
splint --only style/eq-true style/eq-false --autocorrect
```

## Disabling Rules

### In Configuration File

Disable specific rules:

```clojure
{:rules {:style/prefer-condp {:enabled false}}}
```

Disable entire categories:

```clojure
{:rules {:performance {:enabled false}
         :metrics {:enabled false}}}
```

### Inline in Code

Disable for specific forms:

```clojure
; Disable single rule
#_:splint/disable (+ 1 x)

; Disable multiple rules
#_{:splint/disable [style/plus-one lint/redundant-call]}
(do (+ 1 x))
```

**Best practice**: Prefer fixing code over disabling rules. Document why if you must disable.

## CI Integration

Add to GitHub Actions:

```yaml
# .github/workflows/lint.yml
name: Lint

on: [push, pull_request]

jobs:
  splint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install Splint
        run: brew install noahtheduke/tap/splint
      - name: Run Splint
        run: splint -o simple
```

Add to GitLab CI:

```yaml
# .gitlab-ci.yml
lint:
  script:
    - brew install noahtheduke/tap/splint
    - splint -o simple
```

**For CI**: Use simple output format (`-o simple`) for cleaner logs.

## Git Pre-Commit Hook (Optional)

Splint can run automatically before commits. The hook is **disabled by default** to avoid disrupting workflows.

### Setup Instructions

Create `.git/hooks/pre-commit`:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Uncomment to enable Splint pre-commit checks
# echo "Running Splint..."
# if ! splint -q; then
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

**To enable**: Uncomment the lines in the hook script.

**To bypass when enabled**: Use `git commit --no-verify`

### Alternative: Lint Changed Files Only

```bash
#!/usr/bin/env bash
set -euo pipefail

# Get staged Clojure files
FILES=$(git diff --cached --name-only --diff-filter=ACM | grep '\.clj$' || true)

if [ -n "$FILES" ]; then
    echo "Running Splint on staged files..."
    if ! splint -q $FILES; then
        echo "Splint found issues. Commit aborted."
        exit 1
    fi
fi
```

## Command-Line Reference

**Basic usage**:
```bash
splint                          # Lint project
splint src test                 # Lint specific paths
splint -o simple                # Simple output format
splint -q                       # Quiet mode (summary only)
splint --only style             # Run only style rules
splint --autocorrect            # Auto-fix safe issues
splint --auto-gen-config        # Generate baseline config
splint --print-config diff      # Show config changes
```

**For complete command-line documentation, see**: `references/usage.md`

## Best Practices

1. **Run regularly**: Make Splint part of your development workflow, not just CI

2. **Start with defaults**: Run without configuration first to see all issues

3. **Use auto-gen-config**: For existing projects, generate baseline and enable incrementally

4. **Fix incrementally**: Address new issues as they arise rather than fixing all at once

5. **Team alignment**: Discuss and agree on which rules to enable/disable

6. **CI enforcement**: Run Splint in CI to prevent non-idiomatic code from merging

7. **Document exceptions**: If disabling a rule, document why in `.splint.edn`

8. **Combine with other tools**: Use alongside clj-kondo for comprehensive linting

9. **Test after fixing**: Always run tests after applying fixes

10. **Commit before autocorrect**: Never run `--autocorrect` without committing first

11. **Review autocorrect changes**: Manually review all autocorrected changes

12. **Use appropriate output**: Simple for CI, full for development

13. **Lint changed files**: In development, lint only files you're working on

14. **Exclude generated code**: Always exclude auto-generated files in configuration

15. **Performance rules**: Enable performance rules only after team discussion

## Configuration Examples

### Strict Mode - Enable All Rules

```clojure
{:paths ["src" "test"]
 :output {:pattern "full"}
 :rules {:lint {:enabled true}
         :style {:enabled true}
         :naming {:enabled true}
         :performance {:enabled true}
         :metrics {:enabled true
                   :fn-length {:length 15}
                   :parameter-count {:count 4}}}}
```

### Relaxed Mode - Core Rules Only

```clojure
{:paths ["src" "test"]
 :output {:pattern "simple"}
 :rules {:lint {:enabled true}
         :style {:enabled true}
         :naming {:enabled true}
         :performance {:enabled false}
         :metrics {:enabled false}}}
```

### Legacy Codebase - Gradual Adoption

```clojure
{:paths ["src" "test"]
 :output {:pattern "simple"}
 ; Enable only safe, uncontroversial rules
 :rules {:lint/eq-nil {:enabled true}
         :lint/if-else-nil {:enabled true}
         :style/eq-true {:enabled true}
         :style/eq-false {:enabled true}
         :naming/record-name {:enabled true}
         :naming/single-segment-namespace {:enabled true}}
 ; Exclude legacy code
 :global {:excludes ["glob:**/legacy/**"]}}
```

### Team Standards - Naming and Metrics

```clojure
{:paths ["src" "test"]
 :rules {:naming {:enabled true}
         :metrics/fn-length {:enabled true
                            :chosen-style :body
                            :length 20}
         :metrics/parameter-count {:enabled true
                                   :chosen-style :positional
                                   :count 4}}
 :global {:excludes ["glob:**/generated/**"]}}
```

**For more configuration examples, see**: `references/configuration.md`

## Workflow Checklist

### Setting Up Splint

- [ ] Install Splint (Homebrew, Clojure CLI, Leiningen, or Babashka)
- [ ] Verify installation: `splint --version`
- [ ] Create `.splint.edn` configuration file in project root
- [ ] Create `scripts/lint-splint.sh` script
- [ ] Make script executable: `chmod +x scripts/lint-splint.sh`
- [ ] Run initial lint: `./scripts/lint-splint.sh`
- [ ] Add `.splint.edn` to version control
- [ ] Document Splint usage in project README

### For Existing Projects

- [ ] Run `splint --auto-gen-config` to generate baseline
- [ ] Review generated `.splint.edn`
- [ ] Commit baseline configuration
- [ ] Enable one rule or category
- [ ] Run `splint` and fix issues
- [ ] Commit fixes
- [ ] Repeat for each rule/category

### For New Projects

- [ ] Create configuration with core rules enabled
- [ ] Run `splint` as you develop
- [ ] Fix issues immediately
- [ ] Add to CI pipeline
- [ ] Consider pre-commit hook

### Integrating with CI

- [ ] Add Splint installation to CI script
- [ ] Add Splint run to CI script
- [ ] Use simple output format: `-o simple`
- [ ] Ensure CI fails if Splint finds issues
- [ ] Test CI integration with a known issue
- [ ] Document CI setup in project docs

### Using Autocorrect

- [ ] Commit all changes first
- [ ] Run `splint --autocorrect`
- [ ] Review changes: `git diff`
- [ ] Run tests to ensure code still works
- [ ] Review that critical comments weren't lost
- [ ] Commit autocorrect changes
- [ ] Document autocorrect usage for team

## Troubleshooting

### Splint Not Found

**Problem**: `command not found: splint`

**Solution**: Install via Homebrew or add to `deps.edn`/`project.clj` and use `clojure -M:splint` or `lein splint`.

### No Files Processed

**Problem**: Splint processes 0 files

**Solution**: Verify `:paths` in `.splint.edn` or provide paths explicitly: `splint src test`

### Too Many Issues

**Problem**: Hundreds of diagnostics reported

**Solution**: Use `--auto-gen-config` to create baseline, then enable rules incrementally.

### Configuration Not Loading

**Problem**: `.splint.edn` seems ignored

**Solution**: Verify file is in project root, check EDN syntax, use `--print-config local` to debug.

### Autocorrect Breaks Code

**Problem**: Code doesn't work after autocorrect

**Solution**: Always commit before autocorrecting. Review changes carefully. Run tests immediately after.

**For more troubleshooting, see**: `references/usage.md`

## Reference Documentation

Splint has comprehensive reference documentation in the `references/` directory:

- **[rules.md](./references/rules.md)** - Complete documentation of all 100+ linting rules with examples
- **[configuration.md](./references/configuration.md)** - Detailed configuration options and patterns
- **[usage.md](./references/usage.md)** - Command-line usage, integration examples, and best practices

## External Resources

- **Splint GitHub**: https://github.com/NoahTheDuke/splint
- **Clojure Style Guide**: https://guide.clojure.style/
- **Latest Release**: https://github.com/NoahTheDuke/splint/releases
- **cljdoc API Documentation**: https://cljdoc.org/d/io.github.noahtheduke/splint/

## Summary

Splint is a fast, focused linter for Clojure that helps enforce community idioms and style conventions. Key points:

- **Fast**: Processes large codebases in seconds
- **Focused**: Analyzes code shape and idioms, not semantics
- **Configurable**: Enable/disable rules as needed
- **Autocorrect**: Automatically fix many issues (with caution)
- **CI-friendly**: Easy to integrate into continuous integration
- **Incremental**: Adopt gradually with `--auto-gen-config`

Use Splint alongside clj-kondo for comprehensive Clojure linting. Splint focuses on style and idioms, while clj-kondo handles lexical analysis and correctness.
