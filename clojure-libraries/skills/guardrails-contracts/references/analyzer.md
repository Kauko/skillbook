# Guardrails Analyzer Reference

This document covers setup and usage of the Guardrails static analyzer for compile-time type checking.

## Overview

Guardrails Analyzer performs static code analysis to catch type errors and spec violations during development without running code. It provides compile-time validation based on Guardrails specs.

## Prerequisites

- Clojure CLI tools
- Node.js (for ClojureScript analysis)
- Java 21+
- Guardrails library with specs defined

## Installation

### Adding to Project

#### deps.edn

```clojure
{:aliases
 {:analyze
  {:extra-deps {io.github.fulcrologic/guardrails-analyzer
                {:git/url "https://github.com/fulcrologic/guardrails-analyzer"
                 :git/sha "latest-sha-here"}}
   :main-opts ["-m" "com.fulcrologic.guardrails-analyzer.core"]}}}
```

#### Leiningen (project.clj)

```clojure
:profiles
{:dev {:dependencies [[com.fulcrologic/guardrails-analyzer "VERSION"]]}}
```

## Daemon Setup

The analyzer requires a daemon server for IDE integration:

### Building the Daemon

```bash
# Clone repository
git clone https://github.com/fulcrologic/guardrails-analyzer.git
cd guardrails-analyzer

# Build uberjars
make Daemon.jar
```

### Running the Daemon

```bash
# Run from home directory
cd ~
java -jar path/to/Daemon.jar
```

**Important**: The daemon must be running for IDE communication to function.

### Daemon Logs

Logs are available at:
```bash
~/.guardrails/logs
```

## REPL Usage

### Starting the Checker

```clojure
(require '[com.fulcrologic.guardrails-analyzer.checkers.clojure :as cc])

;; Start analyzer
(cc/start {:src-dirs ["src/dev" "src/main"]})
```

### Loading Your Project

**Critical**: Load your project's entry-point namespace before starting the checker:

```clojure
;; Load project first
(require 'myapp.core)

;; Then start checker
(cc/start {:src-dirs ["src/main" "src/dev"]})
```

This ensures all dependencies are loaded and the analyzer has complete context.

### Checker Configuration

```clojure
;; Custom configuration
(cc/start {:src-dirs ["src/main" "src/dev" "src/test"]
           :exclude-dirs ["src/generated"]
           :fail-on-error true})
```

### Stopping the Checker

```clojure
(cc/stop)
```

## IDE Integration

### IntelliJ Plugin

1. Download release from: https://github.com/fulcrologic/guardrails-intellij-plugin
2. Install plugin from disk in IntelliJ (Settings > Plugins > Install from disk)
3. Ensure daemon is running

### Using the Plugin

Trigger analysis via:
1. Open Command Palette (CMD-SHIFT-A on Mac, CTRL-SHIFT-A on Windows/Linux)
2. Type "Check Namespace" or "Guardrails"
3. Select desired action

**Available Actions**:
- Check Current Namespace
- Check Project
- Check File
- Clear Errors

### Plugin Configuration

Configure in IntelliJ settings:

```
Settings > Tools > Guardrails Analyzer
- Daemon Port: 7890 (default)
- Auto-check on save: true/false
- Show warnings: true/false
```

## Command Line Usage

### Basic Analysis

```bash
# Analyze all source files
clojure -M:analyze

# Analyze specific directory
clojure -M:analyze src/main

# Analyze specific file
clojure -M:analyze src/myapp/core.clj
```

### With Options

```bash
# Fail build on violations
clojure -M:analyze --fail-on-error

# Verbose output
clojure -M:analyze --verbose

# Output format
clojure -M:analyze --format json
```

## Configuration File

Create `.guardrails-analyzer.edn` in project root:

```clojure
{:paths ["src/main" "src/dev"]
 :exclude ["src/generated" "src/test"]
 :fail-on-error true
 :checks {:arity true
          :schema-validation true
          :return-type true
          :argument-types true}
 :output {:format :pretty
          :show-context true
          :color true}}
```

## Configuration Options

### `:paths` - Source Directories

```clojure
{:paths ["src/main" "src/dev" "src/test"]}
```

Directories to analyze.

### `:exclude` - Excluded Paths

```clojure
{:exclude ["src/generated" "src/migrations"]}
```

Paths to skip during analysis.

### `:fail-on-error` - Build Failure

```clojure
{:fail-on-error true}
```

Exit with non-zero status if violations found. Useful for CI/CD.

### `:checks` - Analysis Types

```clojure
{:checks {:arity true
          :schema-validation true
          :return-type true
          :argument-types true
          :nil-checking true}}
```

**Options**:
- `:arity` - Check function arity matches specs
- `:schema-validation` - Validate against Malli/spec schemas
- `:return-type` - Check return types match specs
- `:argument-types` - Check argument types
- `:nil-checking` - Detect potential nil pointer errors

### `:output` - Output Configuration

```clojure
{:output {:format :pretty
          :show-context true
          :color true
          :group-by :file}}
```

**Formats**:
- `:pretty` - Human-readable output
- `:json` - Machine-readable JSON
- `:edn` - EDN data format

## Guardrails Modes

### All Mode (Runtime + Static)

```bash
clojure -J-Dguardrails.mode=:all -J-Dguardrails.enabled=true -M:analyze
```

**Features**:
- Runtime contract checking
- Static analysis during compilation
- Full validation

**Use Case**: Development with maximum safety.

### Pro Mode (Static Only)

```bash
clojure -J-Dguardrails.mode=:pro -M:analyze
```

**Features**:
- Static analysis only
- No runtime overhead
- Contracts compiled out

**Use Case**: CI/CD and production builds where runtime checks are not needed.

## CI/CD Integration

### GitHub Actions

```yaml
name: Static Analysis

on: [push, pull_request]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Setup Java
        uses: actions/setup-java@v2
        with:
          java-version: '21'

      - name: Setup Clojure
        uses: DeLaGuardo/setup-clojure@master
        with:
          cli: latest

      - name: Run Guardrails Analyzer
        run: clojure -J-Dguardrails.mode=:pro -M:analyze --fail-on-error
```

### GitLab CI

```yaml
guardrails-analysis:
  stage: test
  image: clojure:openjdk-21
  script:
    - clojure -J-Dguardrails.mode=:pro -M:analyze --fail-on-error
  only:
    - merge_requests
    - main
```

### CircleCI

```yaml
version: 2.1

jobs:
  analyze:
    docker:
      - image: cimg/clojure:1.11
    steps:
      - checkout
      - run:
          name: Guardrails Analysis
          command: clojure -J-Dguardrails.mode=:pro -M:analyze --fail-on-error

workflows:
  version: 2
  build-and-analyze:
    jobs:
      - analyze
```

## Analysis Output

### Pretty Format

```
Analyzing namespace myapp.core...

ERROR in myapp.core/create-user (line 42):
  Contract violation: Argument validation failed
  Expected: [:map [:email string?] [:name string?]]
  Received: [:map [:email string?]]

  Context:
    (create-user {:email "test@example.com"})
             ^-- Missing required key :name

WARNING in myapp.core/get-user (line 58):
  Potential nil return without nil check
  Function: get-user
  Return spec: User
  But may return nil from database query
```

### JSON Format

```json
{
  "results": [
    {
      "level": "error",
      "file": "src/myapp/core.clj",
      "line": 42,
      "column": 15,
      "message": "Contract violation: Argument validation failed",
      "expected": "[:map [:email string?] [:name string?]]",
      "received": "[:map [:email string?]]"
    }
  ],
  "summary": {
    "errors": 1,
    "warnings": 2,
    "files-analyzed": 15
  }
}
```

## Advanced Features

### Path-Based Analysis

The analyzer tracks execution through control flow:

```clojure
(>defn process-user
  [user]
  [User => ProcessedUser]
  (if (:active user)
    (activate-user user)
    (deactivate-user user)))
```

Analyzer validates:
- Both branches return `ProcessedUser`
- All paths through conditionals are type-safe
- Nil handling in `if/when/cond` expressions

### Control Flow Support

Supports analysis through:
- `if`, `when`, `when-not`
- `cond`, `case`
- `and`, `or`
- `let` bindings
- `try/catch` blocks

```clojure
(>defn safe-divide
  [a b]
  [number? number? => [:or number? string?]]
  (if (zero? b)
    "division by zero"
    (/ a b)))

;; Analyzer validates both paths return valid types
```

### Spec Generator Validation

Uses spec generators to validate contracts:

```clojure
(>defn transform-data
  [input]
  [InputSchema => OutputSchema (<- input-generator)]
  (transformation input))

;; Analyzer uses generator to create test cases
```

## Troubleshooting

### Analyzer Not Finding Specs

**Problem**: "No spec found for namespace.foo/bar"

**Solution**:
1. Ensure specs are defined with `>defn`, `>def`, or `>fdef`
2. Load namespace before starting analyzer
3. Check that Guardrails mode is `:all` or `:pro`

```bash
# Verify mode
clojure -J-Dguardrails.mode=:all -e "(System/getProperty \"guardrails.mode\")"
```

### False Positives

**Problem**: Analyzer reports errors for valid code

**Solution**:
1. Check spec accuracy - may need refinement
2. Add type hints if analyzer can't infer types
3. Use exclusions for problematic functions

```clojure
;; Exclude specific function from analysis
(defn ^:guardrails-analyzer/exclude complex-function
  [x]
  ;; Complex logic analyzer struggles with
  )
```

### Performance Issues

**Problem**: Analysis is slow

**Solutions**:
1. Exclude large generated files
2. Analyze incrementally (single namespaces)
3. Use caching (analyzer caches results)

```clojure
{:exclude ["src/generated" "src/migrations"]}
```

### IDE Plugin Not Working

**Checks**:
1. Is daemon running?
2. Check logs at `~/.guardrails/logs`
3. Verify port 7890 is not blocked
4. Restart daemon and IDE

```bash
# Check daemon is running
ps aux | grep Daemon.jar

# View recent logs
tail -f ~/.guardrails/logs/latest.log
```

## Best Practices

1. **Run in CI/CD** - Catch issues before merge
2. **Use Pro Mode in CI** - Static analysis without runtime overhead
3. **Load Project First** - Ensure all dependencies loaded in REPL
4. **Exclude Generated Code** - Focus analysis on hand-written code
5. **Review Warnings** - Not just errors, warnings often indicate issues
6. **Keep Specs Updated** - Analysis quality depends on spec accuracy
7. **Use with clj-kondo** - Complementary tools catch different issues

## Integration with Other Tools

### clj-kondo

Guardrails Analyzer complements clj-kondo:

- **clj-kondo**: Syntax, unused bindings, typos
- **Guardrails Analyzer**: Type checking, contract validation

Use both in CI:

```bash
# First run clj-kondo
clj-kondo --lint src

# Then run guardrails analyzer
clojure -M:analyze --fail-on-error
```

### Eastwood

```bash
# Eastwood for general linting
lein eastwood

# Guardrails for contract validation
clojure -M:analyze
```

### kaocha

Integrate with test runner:

```clojure
;; tests.edn
#kaocha/v1
{:plugins [:kaocha.plugin/guardrails-analyzer]
 :guardrails {:fail-on-error true}}
```

## Documentation Resources

- Main Repository: https://github.com/fulcrologic/guardrails-analyzer
- Architecture Overview: `CLAUDE.md` in repository
- IntelliJ Plugin: https://github.com/fulcrologic/guardrails-intellij-plugin
- Running Tests: `ai/running-tests.md` in repository

## Examples

### Basic Project Setup

```clojure
;; deps.edn
{:paths ["src/main"]

 :aliases
 {:dev {:extra-paths ["src/dev"]
        :extra-deps {com.fulcrologic/guardrails {:mvn/version "1.2.0"}}}

  :analyze {:extra-deps {io.github.fulcrologic/guardrails-analyzer
                         {:git/url "https://github.com/fulcrologic/guardrails-analyzer"
                          :git/sha "latest"}}
            :main-opts ["-m" "com.fulcrologic.guardrails-analyzer.core"]}}}

;; .guardrails-analyzer.edn
{:paths ["src/main"]
 :fail-on-error true
 :checks {:arity true
          :schema-validation true
          :return-type true}}
```

### Running Analysis

```bash
# Development
clojure -J-Dguardrails.mode=:all -M:dev:analyze

# CI/CD
clojure -J-Dguardrails.mode=:pro -M:analyze --fail-on-error
```

## Summary

Guardrails Analyzer provides compile-time validation to catch type errors early:

- **Static Analysis**: Catch errors without running code
- **IDE Integration**: Real-time feedback during development
- **CI/CD**: Fail builds on contract violations
- **Control Flow**: Validates all code paths
- **Zero Runtime Cost**: Pro mode for production builds

Use alongside runtime Guardrails for comprehensive validation throughout the development lifecycle.
