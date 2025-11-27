# Splint Configuration Reference

This document provides comprehensive configuration guidance for Splint.

## Configuration File Format

Splint uses a `.splint.edn` file at the project root to customize behavior. The file format is an EDN map with symbol keys. Configuration must use fully-qualified rule names.

## Basic Configuration Structure

```clojure
{
  ; Paths to lint
  :paths ["src" "test"]

  ; Output format
  :output "full"

  ; Parallel execution
  :parallel true

  ; Rule configuration
  :rules {:lint/eq-nil {:enabled false}
          :performance {:enabled false}
          :metrics/parameter-count {:chosen-style :include-rest
                                    :count 4}}

  ; Global exclusions
  :global {:excludes ["foo" "glob:**/bar.clj" "regex:[a-z].clj"]}
}
```

## Configuration Options

### Output Format

Control how diagnostics are displayed:

```clojure
{:output "full"}  ; Default - full diagnostic with code context
```

**Available formats**:
- `"simple"` - Location, rule name, and message only
- `"full"` - Adds the problematic form and suggested replacement
- `"clj-kondo"` - Compatible format: location, "warning", and message
- `"markdown"` - Full output formatted as markdown with code blocks
- `"json"` - Structured JSON format with all diagnostic details
- `"json-pretty"` - Pretty-printed JSON format

**Example simple output**:
```
src/myapp/core.clj:42:5 [style/prefer-condp] Use condp instead of cond
```

**Example full output**:
```
src/myapp/core.clj:42:5 [style/prefer-condp] Use condp instead of cond
  (cond
    (= x 1) :one
    (= x 2) :two)
=>
  (condp = x
    1 :one
    2 :two)
```

**Example JSON output**:
```json
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
```

### Paths

Specify which directories to lint:

```clojure
{:paths ["src" "test"]}
```

**Default**: If not specified, Splint analyzes directories from `deps.edn` or `project.clj`.

### Parallel Execution

Control parallel processing:

```clojure
{:parallel true}   ; Default - enable parallel execution
{:parallel false}  ; Disable for sequential processing
```

### Quiet and Silent Modes

Control output verbosity:

```clojure
{:quiet true}    ; Suppress diagnostics, show summary only
{:silent true}   ; Print no diagnostics or summary
```

### Summary Display

Control whether summary is shown:

```clojure
{:summary true}   ; Default - show summary
{:summary false}  ; Hide summary
```

### Required Files

Load custom rules from external files:

```clojure
{:require ["custom-rules/my-rules.clj"
           "custom-rules/team-rules.clj"]}
```

## Rule Management

### Enabling/Disabling Individual Rules

```clojure
{:rules {:lint/eq-nil {:enabled false}
         :style/prefer-condp {:enabled true}}}
```

### Disabling Entire Categories

Disable all rules in a genre:

```clojure
{:rules {:performance {:enabled false}
         :metrics {:enabled false}}}
```

This is more efficient than disabling rules individually and prevents new rules in that category from being enabled by default.

### Configuring Rule Options

Many rules support additional configuration options:

```clojure
{:rules {:metrics/fn-length {:enabled true
                             :chosen-style :body
                             :length 15}
         :metrics/parameter-count {:enabled true
                                   :chosen-style :include-rest
                                   :count 3}}}
```

### Rule Configuration Examples by Category

#### Metrics Rules

**Function Length**:
```clojure
{:metrics/fn-length {:enabled true
                     :chosen-style :body  ; or :defn
                     :length 10}}
```

Options:
- `:chosen-style :body` - Count lines from parameter vector onward
- `:chosen-style :defn` - Count entire form including function name
- `:length` - Maximum line count (default: 10)

**Parameter Count**:
```clojure
{:metrics/parameter-count {:enabled true
                           :chosen-style :positional  ; or :include-rest
                           :count 4}}
```

Options:
- `:chosen-style :positional` - Exclude `& args` from count
- `:chosen-style :include-rest` - Include rest parameters in count
- `:count` - Maximum parameter count (default: 4)

#### Performance Rules

**Into Transducer**:
```clojure
{:performance/into-transducer {:enabled true
                               :fn-0-arg #{list vector set}
                               :fn-1-arg #{map filter remove}}}
```

Options:
- `:fn-0-arg` - Set of 0-arity functions to recognize as transducers
- `:fn-1-arg` - Set of 1-arity functions to recognize as transducers

**Single Literal Merge**:
```clojure
{:performance/single-literal-merge {:enabled true
                                    :chosen-style :single}}
```

Options:
- `:chosen-style :dynamic` - Choose best approach based on context
- `:chosen-style :single` - Use single map literal
- `:chosen-style :multiple` - Use chained `assoc` calls

## File Exclusion Patterns

The `:global` configuration option manages file exclusions using `:excludes` with a vector of patterns.

```clojure
{:global {:excludes ["generated"
                     "glob:**/node_modules/**"
                     "regex:.*_test\\.clj$"
                     "re-find:legacy"]}}
```

### Exclusion Pattern Syntax

| Syntax | Behavior | Example |
|--------|----------|---------|
| `glob:pattern` | Uses Java PathMatcher; matches full path | `"glob:**/test/**"` |
| `regex:pattern` | Java regex; matches full path | `"regex:.*_test\\.clj$"` |
| `re-find:pattern` | Java regex; matches partial path | `"re-find:legacy"` |
| `string:pattern` | Fixed string inclusion check | `"string:generated"` |
| (no prefix) | Defaults to `re-find` syntax | `"legacy"` |

### Exclusion Examples

**Exclude all test files**:
```clojure
{:global {:excludes ["glob:**/*_test.clj" "glob:**/*_test.cljs"]}}
```

**Exclude generated code directories**:
```clojure
{:global {:excludes ["glob:**/generated/**" "glob:**/target/**"]}}
```

**Exclude files matching pattern**:
```clojure
{:global {:excludes ["regex:.*\\.generated\\.clj$"]}}
```

**Exclude by substring**:
```clojure
{:global {:excludes ["re-find:legacy" "re-find:deprecated"]}}
```

## Inline Rule Disabling

Disable rules for specific code without modifying configuration files.

### Disable Single Rule

```clojure
; Disable for one form
#_:splint/disable (+ 1 x)
```

### Disable Multiple Rules

```clojure
; Disable multiple rules for one form
#_{:splint/disable [style/plus-one lint/redundant-call]}
(do (+ 1 x))
```

### Disable Specific Rules

```clojure
; Disable only style rules
#_{:splint/disable [style/prefer-condp style/when-do]}
(cond
  (= x 1) :one
  (= x 2) :two)
```

## Configuration Precedence

Options merge with this priority (highest to lowest):

1. **Inline code disabling** - `#_:splint/disable` annotations
2. **Command-line arguments** - Flags like `--output`, `--only`
3. **`.splint.edn` file settings** - Local configuration file
4. **Defaults** - Built-in default settings

### Example

Given this configuration:

```clojure
; .splint.edn
{:output "simple"
 :rules {:style/prefer-condp {:enabled true}}}
```

And this command:

```bash
splint --output json
```

The effective configuration will be:
- Output: `json` (command-line overrides config file)
- Rule: `style/prefer-condp` enabled (from config file)

## Auto-Generated Configuration

Use the `--auto-gen-config` flag to generate a baseline `.splint.edn` file:

```bash
splint --auto-gen-config
```

This creates a configuration file that:
- **Disables all currently failing rules**
- Includes diagnostics count for each rule
- Provides rule descriptions
- Lists available configuration styles

### Example Auto-Generated Config

```clojure
{:rules
 {; 15 diagnostics
  ; Use condp instead of multiple cond predicates
  ; Configuration options: :chosen-style #{:safe :aggressive}
  :style/prefer-condp {:enabled false}

  ; 8 diagnostics
  ; Use clojure.string functions instead of Java interop
  :style/prefer-clj-string {:enabled false}

  ; 3 diagnostics
  ; Functions should be fewer than 10 lines
  ; Configuration options: :chosen-style #{:body :defn}, :length (number)
  :metrics/fn-length {:enabled false}}}
```

### Using Auto-Generated Config

1. Generate baseline: `splint --auto-gen-config`
2. Review the configuration file
3. Selectively enable rules one at a time
4. Fix issues as you enable each rule
5. Commit the configuration with your fixes

This approach allows incremental adoption of Splint without overwhelming your codebase with fixes.

## Print Configuration

View your effective configuration:

```bash
splint --print-config TYPE
```

**Types**:
- `diff` - Show only changes from defaults
- `local` - Show loaded config file contents
- `full` - Show merged defaults with custom config

### Example Output

**diff** - Only customizations:
```clojure
{:output "simple"
 :rules {:style/prefer-condp {:enabled false}}}
```

**local** - Contents of `.splint.edn`:
```clojure
{:paths ["src" "test"]
 :output "simple"
 :rules {:style/prefer-condp {:enabled false}
         :performance {:enabled false}}}
```

**full** - Complete merged configuration:
```clojure
{:paths ["src" "test"]
 :output "simple"
 :parallel true
 :quiet false
 :silent false
 :summary true
 :rules {:lint/eq-nil {:enabled true}
         :lint/if-else-nil {:enabled true}
         ; ... all rules with effective settings
         :style/prefer-condp {:enabled false}
         :performance {:enabled false}}}
```

## Common Configuration Patterns

### Strict Mode - Enable All Rules

```clojure
{:paths ["src" "test"]
 :output "full"
 :rules {:lint {:enabled true}
         :style {:enabled true}
         :naming {:enabled true}
         :performance {:enabled true}
         :metrics {:enabled true}}}
```

### Relaxed Mode - Style and Lint Only

```clojure
{:paths ["src" "test"]
 :output "simple"
 :rules {:lint {:enabled true}
         :style {:enabled true}
         :naming {:enabled false}
         :performance {:enabled false}
         :metrics {:enabled false}}}
```

### CI Mode - Fail Fast

```clojure
{:paths ["src" "test"]
 :output "simple"
 :quiet false
 :summary true
 :rules {:lint {:enabled true}
         :style {:enabled true}
         :naming {:enabled true}}}
```

### Development Mode - Warnings Only

```clojure
{:paths ["src" "test"]
 :output "full"
 :quiet false
 :rules {:lint {:enabled true}
         :style {:enabled true}
         ; Disable contentious rules
         :style/prefer-condp {:enabled false}
         :metrics {:enabled false}
         :performance {:enabled false}}}
```

### Legacy Codebase - Gradual Adoption

```clojure
{:paths ["src" "test"]
 :output "simple"
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

### Performance-Focused

```clojure
{:paths ["src"]
 :output "full"
 :rules {:performance {:enabled true}
         :performance/assoc-many {:enabled true}
         :performance/get-in-literals {:enabled true}
         :performance/get-keyword {:enabled true}
         :performance/into-transducer {:enabled true}
         :performance/single-literal-merge {:enabled true
                                            :chosen-style :multiple}}}
```

### Team Standards Enforcement

```clojure
{:paths ["src" "test"]
 :output "full"
 :rules {:naming {:enabled true}
         :naming/conventional-aliases {:enabled true}
         :naming/lisp-case {:enabled true}
         :naming/predicate {:enabled true}
         :metrics/fn-length {:enabled true
                            :chosen-style :body
                            :length 20}
         :metrics/parameter-count {:enabled true
                                   :chosen-style :positional
                                   :count 4}}
 :global {:excludes ["glob:**/generated/**"]}}
```

## Configuration Best Practices

1. **Start with defaults**: Run Splint without configuration first to see all issues

2. **Use auto-gen-config**: Generate baseline configuration with `--auto-gen-config`

3. **Enable incrementally**: Enable rules one category at a time

4. **Document exceptions**: Add comments explaining why rules are disabled

5. **Use inline disabling sparingly**: Prefer fixing code over disabling rules

6. **Exclude generated code**: Always exclude auto-generated files

7. **Align with team**: Discuss and agree on which rules to enable

8. **Review regularly**: Periodically review disabled rules to see if they can be enabled

9. **Use version control**: Commit `.splint.edn` to share configuration across team

10. **Test in CI**: Run Splint in continuous integration to enforce standards

## Troubleshooting Configuration

### Configuration Not Loading

**Problem**: Splint ignores your `.splint.edn` file.

**Solutions**:
- Verify file is in project root (same directory as `deps.edn` or `project.clj`)
- Check file uses EDN syntax (not JSON or other format)
- Ensure rule names use fully-qualified format (`:genre/rule-name`)
- Verify file is readable (check permissions)

### Rules Still Triggering After Disabling

**Problem**: Disabled rules still report diagnostics.

**Solutions**:
- Check rule name spelling (`:style/prefer-condp` not `:style/prefer-cond-p`)
- Use fully-qualified names (`:style/eq-true` not `:eq-true`)
- Verify configuration loaded with `--print-config local`
- Check if command-line flags override config file
- Ensure proper EDN map syntax with colons

### Exclusions Not Working

**Problem**: Excluded files still being linted.

**Solutions**:
- Add appropriate prefix (`glob:`, `regex:`, `re-find:`, or `string:`)
- Check glob pattern syntax (use `**` for recursive directories)
- Verify path separators match your OS
- Test regex patterns separately before adding to config
- Use `--print-config full` to see effective configuration

### Performance Issues

**Problem**: Splint runs slowly on large codebase.

**Solutions**:
- Ensure `:parallel true` is set (default)
- Exclude unnecessary paths (build outputs, dependencies)
- Disable expensive rules (like metrics rules on large files)
- Use `:quiet true` to reduce output overhead
- Consider linting only changed files in development

## Configuration Schema

For reference, here's the complete configuration schema:

```clojure
{; Optional: Paths to lint
 :paths [string ...]

 ; Optional: Output format
 :output string  ; "simple" | "full" | "clj-kondo" | "markdown" | "json" | "json-pretty"

 ; Optional: Parallel execution
 :parallel boolean

 ; Optional: Quiet mode
 :quiet boolean

 ; Optional: Silent mode
 :silent boolean

 ; Optional: Show summary
 :summary boolean

 ; Optional: Required files
 :require [string ...]

 ; Optional: Rule configuration
 :rules {keyword/symbol {; Enable/disable rule
                         :enabled boolean

                         ; Rule-specific options
                         :chosen-style keyword
                         :length number
                         :count number
                         :fn-0-arg set
                         :fn-1-arg set
                         ; ... other rule-specific options
                         }

         ; Category-level enable/disable
         keyword {:enabled boolean}}

 ; Optional: Global settings
 :global {; File exclusion patterns
          :excludes [string ...]}}
```

## Further Reading

- [Rules Reference](./rules.md) - Complete documentation of all rules
- [Usage Guide](./usage.md) - Command-line usage and examples
- [GitHub Repository](https://github.com/NoahTheDuke/splint) - Source code and issues
