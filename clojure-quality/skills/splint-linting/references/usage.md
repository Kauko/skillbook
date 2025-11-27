# Splint Usage Reference

This document provides comprehensive usage guidance for Splint command-line interface.

## Installation

### Clojure CLI

Add to `deps.edn`:

```clojure
:aliases {:splint {:extra-deps {io.github.noahtheduke/splint {:mvn/version "1.22.0"}}
                   :main-opts ["-m" "noahtheduke.splint"]}}
```

Run with:
```bash
clojure -M:splint [args...]
```

### Leiningen

Add to `project.clj`:

```clojure
:profiles {:dev {:dependencies [[io.github.noahtheduke/splint "1.22.0"]]}}
:aliases {"splint" ["run" "-m" "noahtheduke.splint"]}
```

Run with:
```bash
lein splint [args...]
```

### Babashka

Requires version 1.12.205+. Add to `bb.edn`:

```clojure
:tasks {splint {:extra-deps {io.github.noahtheduke/splint {:mvn/version "1.22.0"}}
                :task noahtheduke.splint/-main}}
```

Run with:
```bash
bb splint [args...]
```

### Homebrew (Binary Installation)

macOS and Linux users can install via Homebrew:

```bash
brew install noahtheduke/tap/splint
```

Verify installation:
```bash
splint --version
```

### Binary Download

Download the latest release from:
https://github.com/NoahTheDuke/splint/releases

Extract and add to your PATH.

## Basic Command Structure

Splint can be invoked with or without file paths:

```bash
# Lint project using paths from deps.edn or project.clj
splint

# Lint specific paths
splint src test

# Use -- separator after options
splint [options] -- [path...]
```

**Default behavior**: When no paths are provided, Splint analyzes directories from `deps.edn` or `project.clj`.

## Command-Line Options

### Output Format

Control how diagnostics are displayed:

```bash
splint -o simple           # Location, rule, message only
splint -o full             # Default: adds code context and suggestions
splint -o clj-kondo        # clj-kondo compatible format
splint -o markdown         # Markdown formatted output
splint -o json             # Structured JSON output
splint -o json-pretty      # Pretty-printed JSON
```

**Aliases**: `-o`, `--output`

### Require Custom Rules

Load custom rules from external files:

```bash
splint -r custom-rules/team-rules.clj
splint --require custom-rules/rules1.clj --require custom-rules/rules2.clj
```

**Aliases**: `-r`, `--require`

**Note**: Can be specified multiple times to load multiple rule files.

### Only Specific Rules

Execute only specified rule(s) or genre(s):

```bash
splint --only style/prefer-condp
splint --only style                    # Run all style rules
splint --only style naming              # Run style and naming rules
splint --only style/eq-true style/eq-false
```

### Parallel Execution

Control parallel processing:

```bash
splint --parallel          # Enable (default)
splint --no-parallel       # Disable for sequential processing
```

### Autocorrect

Automatically apply safe changes:

```bash
splint --autocorrect
```

**Warning**: Autocorrect removes all comments and uneval blocks. Always commit your code before running with this flag.

**Recommended workflow**:
1. Commit your changes
2. Run `splint --autocorrect`
3. Review the changes carefully
4. Test that code still works
5. Commit the fixes

### Quiet Mode

Suppress diagnostics, show summary only:

```bash
splint -q
splint --quiet
```

**Output**: Only shows summary statistics, not individual diagnostics.

### Silent Mode

Print no diagnostics or summary:

```bash
splint -s
splint --silent
```

**Use case**: Exit code only, no output. Useful for scripts that only need pass/fail status.

### Summary Control

Control whether summary is displayed:

```bash
splint --summary           # Enable (default)
splint --no-summary        # Disable
```

### Show Errors Only

Display only parsing or internal errors:

```bash
splint --errors
```

**Use case**: Troubleshooting Splint itself or finding files that can't be parsed.

### Print Configuration

Display configuration path and contents:

```bash
splint --print-config diff    # Show only changes from defaults
splint --print-config local   # Show loaded config file contents
splint --print-config full    # Show merged defaults with custom config
```

### Auto-Generate Configuration

Create a `.splint.edn` file that disables all currently failing rules:

```bash
splint --auto-gen-config
```

This creates a baseline configuration file with:
- All failing rules disabled
- Diagnostic counts for each rule
- Rule descriptions
- Available configuration options

**Workflow**:
1. Run `splint --auto-gen-config` to create baseline
2. Review the generated `.splint.edn`
3. Enable rules incrementally
4. Fix issues as you enable each rule

### Help

Display command-line options:

```bash
splint -h
splint --help
```

### Version

Display current version:

```bash
splint -v
splint --version
```

## Output Formats in Detail

### Simple Format

Minimal output showing location, rule, and message:

```
src/myapp/core.clj:42:5 [style/prefer-condp] Use condp instead of cond
src/myapp/util.clj:15:3 [lint/redundant-let] Remove unnecessary let binding
src/myapp/db.clj:88:7 [style/prefer-clj-string] Use clojure.string/lower-case
```

**Use case**: Quick overview, CI logs, scripts.

### Full Format (Default)

Adds problematic code and suggested replacement:

```
src/myapp/core.clj:42:5 [style/prefer-condp] Use condp instead of cond
  (cond
    (= x 1) :one
    (= x 2) :two
    (= x 3) :three)
=>
  (condp = x
    1 :one
    2 :two
    3 :three)
```

**Use case**: Development, detailed diagnostics, learning.

### clj-kondo Format

Compatible with clj-kondo output format:

```
src/myapp/core.clj:42:5: warning: Use condp instead of cond
```

**Use case**: Integration with tools expecting clj-kondo output.

### Markdown Format

Full output formatted as markdown with code blocks:

````markdown
## src/myapp/core.clj:42:5

**Rule**: style/prefer-condp

**Message**: Use condp instead of cond

**Current**:
```clojure
(cond
  (= x 1) :one
  (= x 2) :two)
```

**Suggested**:
```clojure
(condp = x
  1 :one
  2 :two)
```
````

**Use case**: Documentation, reports, PR comments.

### JSON Format

Structured JSON output:

```json
[
  {
    "rule-name": "style/prefer-condp",
    "form": "(cond (= x 1) :one (= x 2) :two)",
    "message": "Use condp instead of cond",
    "alt": "(condp = x 1 :one 2 :two)",
    "line": 42,
    "column": 5,
    "end-line": 44,
    "end-column": 10,
    "filename": "src/myapp/core.clj"
  }
]
```

**Use case**: Machine processing, custom tooling, analysis.

### JSON Pretty Format

Pretty-printed JSON for readability:

```json
[
  {
    "rule-name": "style/prefer-condp",
    "form": "(cond (= x 1) :one (= x 2) :two)",
    "message": "Use condp instead of cond",
    "alt": "(condp = x 1 :one 2 :two)",
    "line": 42,
    "column": 5,
    "end-line": 44,
    "end-column": 10,
    "filename": "src/myapp/core.clj"
  }
]
```

**Use case**: Human-readable JSON, debugging integrations.

## Common Usage Patterns

### Development Workflow

**Quick check during development**:
```bash
splint src/myapp/core.clj
```

**Check all modified files**:
```bash
git diff --name-only | grep '\.clj' | xargs splint
```

**Full project lint**:
```bash
splint
```

### CI Integration

**Basic CI check**:
```bash
splint || exit 1
```

**With simple output for logs**:
```bash
splint -o simple
```

**Quiet mode with exit code only**:
```bash
splint -q || { echo "Lint failed"; exit 1; }
```

**GitHub Actions example**:
```yaml
- name: Install Splint
  run: brew install noahtheduke/tap/splint

- name: Run Splint
  run: splint -o simple
```

**GitLab CI example**:
```yaml
lint:
  script:
    - brew install noahtheduke/tap/splint
    - splint -o simple
```

### Pre-commit Hook

**Basic pre-commit hook**:
```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Running Splint..."
if ! splint -q; then
    echo "Splint found issues. Commit aborted."
    echo "Fix issues or use 'git commit --no-verify' to skip."
    exit 1
fi
```

**Hook for changed files only**:
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

### Incremental Adoption

**Step 1: Generate baseline config**:
```bash
splint --auto-gen-config
```

**Step 2: Review generated config**:
```bash
cat .splint.edn
```

**Step 3: Enable one rule**:
Edit `.splint.edn` to enable a single rule, then:
```bash
splint
```

**Step 4: Fix issues for that rule**:
Fix all issues, then commit.

**Step 5: Repeat**:
Enable another rule and repeat.

### Specific Rule Focus

**Check only style issues**:
```bash
splint --only style
```

**Check naming conventions**:
```bash
splint --only naming
```

**Check performance opportunities**:
```bash
splint --only performance
```

**Check specific rules**:
```bash
splint --only style/prefer-condp style/prefer-clj-string
```

### Autocorrect Workflow

**Full autocorrect** (DANGEROUS - always commit first):
```bash
git add .
git commit -m "Before autocorrect"
splint --autocorrect
# Review changes carefully
git diff
# Test your code
lein test
# If good, commit
git add .
git commit -m "Apply Splint autocorrect"
```

**Autocorrect specific rules only**:
```bash
splint --only style/eq-true style/eq-false --autocorrect
```

### Configuration Management

**Check current config**:
```bash
splint --print-config local
```

**See what's different from defaults**:
```bash
splint --print-config diff
```

**See full effective config**:
```bash
splint --print-config full
```

**Validate config loads correctly**:
```bash
splint --print-config local > /dev/null && echo "Config OK"
```

## Exit Codes

Splint uses standard Unix exit codes:

- **0**: Success, no diagnostics found
- **1**: Diagnostics found, or error occurred
- **Non-zero**: Error or issues detected

**Example usage in scripts**:
```bash
if splint -q; then
    echo "No issues found"
else
    echo "Issues detected"
    exit 1
fi
```

## Performance Considerations

### Parallel Execution

Splint enables parallel execution by default, which significantly speeds up linting large codebases.

**Benchmark**: On a 223-file project, Splint completed in ~6 seconds (vs. Kibit's 34+ minutes).

**Disable if needed**:
```bash
splint --no-parallel
```

### Limiting Scope

**Lint only changed files**:
```bash
git diff --name-only main | grep '\.clj' | xargs splint
```

**Lint specific directories**:
```bash
splint src/critical test/critical
```

**Use exclusions** in `.splint.edn`:
```clojure
{:global {:excludes ["glob:**/generated/**"
                     "glob:**/target/**"
                     "glob:**/node_modules/**"]}}
```

### Output Optimization

**Reduce output overhead**:
```bash
splint -o simple -q   # Minimal output
```

**Silent mode for exit code only**:
```bash
splint -s
```

## Troubleshooting

### Splint Not Found

**Problem**: `command not found: splint`

**Solutions**:
- Install via Homebrew: `brew install noahtheduke/tap/splint`
- Or use Clojure CLI: `clojure -M:splint` (requires alias in `deps.edn`)
- Or use Leiningen: `lein splint` (requires alias in `project.clj`)

### No Files Processed

**Problem**: Splint processes 0 files

**Solutions**:
- Verify paths exist: `ls src test`
- Check `.splint.edn` `:paths` configuration
- Ensure you're in project root directory
- Verify `deps.edn` or `project.clj` exists if not specifying paths

### Too Many Diagnostics

**Problem**: Splint reports hundreds of issues

**Solutions**:
- Use `--auto-gen-config` to create baseline
- Enable rules incrementally
- Disable contentious rules
- Focus on one category at a time with `--only`

### Autocorrect Breaks Code

**Problem**: Code doesn't work after `--autocorrect`

**Solutions**:
- Always commit before autocorrecting
- Review changes with `git diff`
- Autocorrect may remove comments - check critical documentation
- Run tests immediately after autocorrecting
- Consider enabling only safe rules for autocorrect

### Slow Performance

**Problem**: Splint takes too long

**Solutions**:
- Ensure `--parallel` is enabled (default)
- Exclude unnecessary paths
- Lint only changed files in development
- Check if disk I/O is bottleneck
- Consider disabling expensive rules

### Configuration Not Loading

**Problem**: `.splint.edn` seems ignored

**Solutions**:
- Verify file is in project root
- Check EDN syntax is valid
- Use `--print-config local` to see if it loads
- Ensure rule names use correct format (`:genre/rule-name`)
- Check file permissions

## Integration Examples

### Makefile

```makefile
.PHONY: lint
lint:
	splint -o simple

.PHONY: lint-fix
lint-fix:
	splint --autocorrect
```

### npm scripts (for ClojureScript projects)

```json
{
  "scripts": {
    "lint": "splint -o simple",
    "lint:fix": "splint --autocorrect"
  }
}
```

### Justfile

```just
lint:
    splint -o simple

lint-fix:
    splint --autocorrect

lint-changed:
    git diff --name-only main | grep '\.clj' | xargs splint
```

### Editor Integration

**VS Code** (via shell command):
```json
{
  "tasks": [
    {
      "label": "Splint",
      "type": "shell",
      "command": "splint -o full ${file}",
      "problemMatcher": []
    }
  ]
}
```

**Emacs** (via compile command):
```elisp
(defun run-splint ()
  "Run Splint on current file."
  (interactive)
  (compile (format "splint %s" buffer-file-name)))
```

**Vim** (via makeprg):
```vim
:set makeprg=splint\ %
:make
```

## Best Practices

1. **Install as binary**: Homebrew installation is fastest and easiest for development

2. **Run frequently**: Lint during development, not just in CI

3. **Start with defaults**: Run without configuration first to see all issues

4. **Use auto-gen-config**: Generate baseline configuration for existing projects

5. **Enable incrementally**: Turn on rules one at a time for existing codebases

6. **Commit before autocorrect**: Always commit before using `--autocorrect`

7. **Review autocorrect changes**: Manually review all autocorrected changes

8. **Use in CI**: Enforce standards by running Splint in continuous integration

9. **Configure per-project**: Each project should have its own `.splint.edn`

10. **Document exceptions**: Add comments explaining why rules are disabled

11. **Lint changed files**: In development, lint only files you're working on

12. **Use appropriate output**: Simple for CI, full for development

13. **Combine with other tools**: Use alongside clj-kondo, Eastwood, etc.

14. **Test after fixing**: Always run tests after fixing lint issues

15. **Team alignment**: Discuss and agree on rule configuration with team

## Further Reading

- [Rules Reference](./rules.md) - Complete documentation of all rules
- [Configuration Reference](./configuration.md) - Detailed configuration options
- [Splint GitHub](https://github.com/NoahTheDuke/splint) - Source code and issues
- [Clojure Style Guide](https://guide.clojure.style/) - Community style conventions
