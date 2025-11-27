# clj-kondo Configuration Reference

This document provides comprehensive configuration options for clj-kondo. For workflow guidance, see the main skill file.

## Configuration File Locations

clj-kondo reads configuration from multiple sources, merging them in this order (later overrides earlier):

1. **Home directory:** `~/.config/clj-kondo/config.edn` (respects `XDG_CONFIG_HOME`)
2. **Config paths:** Referenced via `:config-paths` in project config
3. **Project config:** `.clj-kondo/config.edn` at project root
4. **Environment variable:** `CLJ_KONDO_EXTRA_CONFIG_DIR`
5. **Command line:** `--config` flag
6. **Namespace-level:** Via `ns` metadata

Use `^:replace` metadata to replace configuration sections instead of merging.

## Core Configuration Structure

```clojure
{:linters {...}           ;; Linter-specific settings
 :output {...}            ;; Output formatting
 :lint-as {...}           ;; Macro expansion rules
 :hooks {...}             ;; Custom hook registration
 :exclude-files [...]     ;; File exclusion patterns
 :skip-comments true      ;; Ignore (comment ...) forms
 :config-in-ns {...}      ;; Namespace-level config
 :config-in-call {...}    ;; Context-specific config
 :ns-groups [...]}        ;; Namespace grouping
```

## Linter Control

### Basic Linter Configuration

```clojure
{:linters {:invalid-arity {:level :error}
           :unused-binding {:level :warning}
           :redundant-do {:level :off}}}
```

**Levels:** `:error`, `:warning`, `:info`, `:off`

### Short Notation (2023.03.17+)

```clojure
;; Ignore specific linters
{:ignore [:unresolved-symbol :invalid-arity]}

;; Suppress all linters
{:ignore true}
```

### Per-Linter Exclusions

```clojure
{:linters {:unused-namespace {:level :warning
                               :exclude [clojure.test
                                        my.test.utils]}
           :private-call {:exclude [my.internal/dev-fn]}
           :unused-binding {:exclude-destructured-as true}}}
```

### Optional Linters (Disabled by Default)

Enable these explicitly when needed:

```clojure
{:linters {:missing-docstring {:level :warning}
           :unsorted-required-namespaces {:level :warning}
           :refer {:level :warning}
           :single-key-in {:level :warning}
           :shadowed-var {:level :warning}
           :docstring-no-summary {:level :warning}
           :docstring-leading-trailing-whitespace {:level :warning}
           :used-underscored-binding {:level :warning}
           :warn-on-reflection {:level :warning}
           :unused-value {:level :warning}
           :redundant-call {:level :warning}
           :redundant-str-call {:level :warning}}}
```

## Namespace-Level Configuration

### Via Config File

```clojure
{:config-in-ns {my.namespace {:linters {:unresolved-symbol {:level :off}}}
                my.test.ns {:ignore [:missing-docstring]}}}
```

### Via Namespace Metadata

```clojure
(ns my.namespace
  {:clj-kondo/config '{:linters {:unresolved-symbol {:level :off}}}}
  (:require [clojure.string :as str]))
```

## Expression-Level Suppression

Suppress warnings before specific expressions:

```clojure
;; Suppress all warnings
#_:clj-kondo/ignore
(inc 1 2 3)

;; Suppress specific linters
#_{:clj-kondo/ignore [:invalid-arity]}
(inc 1 2 3)

;; Suppress multiple linters
#_{:clj-kondo/ignore [:invalid-arity :unused-binding]}
(let [x 1] (inc 1 2 3))
```

## Macro Configuration

### Lint-As Configuration

Teach clj-kondo to lint custom macros like built-in ones:

```clojure
{:lint-as {my.lib/defroute clojure.core/def
           my.lib/with-db clojure.core/let
           my.lib/defcomponent clojure.core/defn
           my.lib/>defn clojure.core/defn}}
```

**Built-in lint-as targets:**
- `clojure.core/let` - For binding forms
- `clojure.core/def` - For top-level definitions
- `clojure.core/defn` - For function definitions
- `clojure.core/defmethod` - For multimethod implementations
- `clj-kondo.lint-as/def-catch-all` - For complex def forms

### Via Macro Metadata (2023.03.17+)

```clojure
(defmacro with-foo
  {:clj-kondo/lint-as 'clojure.core/let}
  [bindings & body]
  `(let ~bindings ~@body))
```

### Common Library Configurations

```clojure
{:lint-as {;; Malli
           malli.core/def schema.core/defschema
           malli.core/defn schema.core/defn

           ;; Compojure
           compojure.core/defroutes clojure.core/def
           compojure.core/GET clojure.core/def
           compojure.core/POST clojure.core/def
           compojure.core/PUT clojure.core/def
           compojure.core/DELETE clojure.core/def
           compojure.core/context clojure.core/def

           ;; Core.async
           clojure.core.async/go-loop clojure.core/loop

           ;; Test.check
           clojure.test.check.properties/for-all clojure.core/let

           ;; Criterium
           criterium.core/bench clojure.core/let
           criterium.core/quick-bench clojure.core/let

           ;; Integrant
           integrant.core/defmethod-ig clojure.core/defmethod

           ;; Guardrails
           guardrails.core/defn clojure.core/defn
           guardrails.core/>defn clojure.core/defn
           guardrails.core/>defn- clojure.core/defn-}}
```

## Context-Specific Configuration

### Config in Call

Apply configuration within specific function/macro calls:

```clojure
{:config-in-call {my.ns/ignore-issues {:linters {:unresolved-symbol {:level :off}}}
                  my.test/with-mocks {:linters {:private-call {:level :off}}}}}
```

### Config in Tag

Apply configuration within reader-tagged literals:

```clojure
{:config-in-tag {jsx {:linters {:unresolved-symbol {:level :error}}}}}
```

### Config in Comment

Special rules for comment forms:

```clojure
{:config-in-comment {:linters {:unresolved-namespace {:level :off}
                               :unresolved-symbol {:level :off}}}}
```

## Output Configuration

```clojure
{:output {:format :json              ;; :edn, :json, :sarif, :text (default)
          :pattern "custom pattern"  ;; Custom format string
          :progress true             ;; Show progress bar
          :canonical-paths true      ;; Use absolute paths
          :linter-name true          ;; Include linter name in output
          :langs true                ;; Show language context in .cljc
          :include-files ["regex"]   ;; Only include matching files
          :exclude-files ["regex"]}} ;; Exclude matching files
```

### Custom Output Patterns

Template variables for `:pattern`:
- `{{filename}}` - File path
- `{{row}}`, `{{end-row}}` - Line numbers
- `{{col}}`, `{{end-col}}` - Column numbers
- `{{level}}` - Lowercase level (error, warning)
- `{{LEVEL}}` - Uppercase level (ERROR, WARNING)
- `{{message}}` - Lint message
- `{{type}}` - Linter type

**Example patterns:**

```clojure
;; GitHub Actions format
{:output {:pattern "::{{level}} file={{filename}},line={{row}},col={{col}}::{{message}}"}}

;; Machine-readable format
{:output {:pattern "{{filename}}:{{row}}:{{col}}:{{level}}:{{type}}:{{message}}"}}

;; Simple format
{:output {:pattern "{{filename}}:{{row}}: {{message}}"}}
```

## Namespace Groups

Group namespaces for unified configuration:

```clojure
{:ns-groups [{:pattern "foo\\..*" :name foo-group}
             {:pattern "bar\\..*" :name bar-group}
             {:filename-pattern ".*_test\\.clj$" :name test-group}]}
```

Use groups in configuration:

```clojure
{:config-in-ns {foo-group {:linters {:missing-docstring {:level :off}}}}
 :linters {:discouraged-namespace {:groups {foo-group {:level :warning}}}}}
```

## File Handling

### Exclude Files

```clojure
{:exclude-files ["resources/.*"
                 "generated/.*"
                 ".*\\.cljc$"]}  ;; Regex patterns
```

### Auto-Load Configs

```clojure
{:auto-load-configs false}  ;; Disable loading .clj-kondo/*/*/config.edn
```

### Skip Comments

```clojure
{:skip-comments true}  ;; Don't lint code inside (comment ...) forms
```

## Clojure Variant Handling

Control which variants to lint in .cljc files:

```clojure
{:cljc {:features [:clj]}}      ;; Only lint Clojure code
{:cljc {:features [:cljs]}}     ;; Only lint ClojureScript code
{:cljc {:features [:clj :cljs]}} ;; Lint both (default)
```

## Library Configuration Import/Export

### Exporting Configs (Library Authors)

Place config at:
```
clj-kondo.exports/<org>/<libname>/config.edn
```

Example structure:
```
my-library/
└── clj-kondo.exports/
    └── acme/
        └── my-library/
            └── config.edn
```

### Importing Configs (Users)

```bash
# Import all library configs from classpath
clj-kondo --lint "$(clojure -Spath)" --dependencies --parallel --copy-configs

# Skip linting, only copy configs
clj-kondo --lint "$(clojure -Spath)" --copy-configs --skip-lint
```

Requires `.clj-kondo/` directory to exist.

## Advanced Configuration Examples

### Project-Wide Strictness

```clojure
{:linters {:unresolved-symbol {:level :error}
           :unresolved-namespace {:level :error}
           :unresolved-var {:level :error}
           :invalid-arity {:level :error}
           :type-mismatch {:level :error}
           :private-call {:level :error}
           :missing-else-branch {:level :warning}
           :unused-binding {:level :warning}
           :unused-namespace {:level :warning}
           :unused-private-var {:level :warning}
           :redundant-do {:level :warning}
           :redundant-let {:level :warning}}}
```

### Test-Specific Configuration

```clojure
{:ns-groups [{:pattern ".*-test$" :name tests}]
 :config-in-ns {tests {:linters {:missing-docstring {:level :off}
                                 :private-call {:level :off}}}}}
```

### Development vs Production

**Development:** `.clj-kondo/config.edn`
```clojure
{:linters {:unused-binding {:level :warning}}}
```

**CI:** Override via environment
```bash
export CLJ_KONDO_EXTRA_CONFIG_DIR=/path/to/strict-config
clj-kondo --lint src
```

## Hook Configuration

Register custom hooks:

```clojure
{:hooks {:analyze-call {my.ns/custom-macro hooks.custom/analyze}
         :macroexpand {my.ns/simple-macro hooks.custom/expand}}}
```

See `references/hooks.md` for detailed hook documentation.

## Performance Tuning

```clojure
{:output {:exclude-files ["generated/.*"]}  ;; Skip generated files
 :skip-comments true                        ;; Skip comment forms
 :auto-load-configs false}                  ;; Disable auto-loading
```

Command-line options:
```bash
# Use parallel processing
clj-kondo --lint src --parallel

# Disable cache
clj-kondo --lint src --cache false
```

## Debugging Configuration

Print effective configuration:
```bash
clj-kondo --config '{:output {:format :edn}}' --lint - <<< "(+ 1 2 3)"
```

Clear cache when troubleshooting:
```bash
rm -rf .clj-kondo/.cache
```

## References

- Default configuration: [clj-kondo config.clj](https://github.com/clj-kondo/clj-kondo/blob/master/src/clj_kondo/impl/config.clj)
- Example configs: [clj-kondo's own config](https://github.com/clj-kondo/clj-kondo/blob/master/.clj-kondo/config.edn)
- Community configs: [clj-kondo/config](https://github.com/clj-kondo/config) repository
