# Guardrails Configuration Reference

This document covers all configuration options for Guardrails in development and production environments.

## Enabling Guardrails

### JVM Configuration

Enable Guardrails via JVM system property:

```bash
# Enable guardrails
clojure -J-Dguardrails.enabled=true

# Disable guardrails
clojure -J-Dguardrails.enabled=false
```

### ClojureScript Configuration

For shadow-cljs projects, add to `shadow-cljs.edn`:

```clojure
{:builds
 {:app
  {:target :browser
   :dev {:compiler-options
         {:external-config {:guardrails {}}}}
   :release {:compiler-options
             {:external-config {:guardrails {:enabled false}}}}}}}
```

### Default Behavior

**Important**: Guardrails is disabled by default to prevent accidental production overhead. You must explicitly enable it in development.

## Configuration File: guardrails.edn

Create a `guardrails.edn` file in your project root or resources directory:

```clojure
{:throw? true
 :guardrails/mcps 100
 :guardrails/stack-trace :prune
 :guardrails/compact? true
 :guardrails/trace? true
 :guardrails/use-stderr? false
 :expound {:show-valid-values? true
           :print-specs? true}}
```

## Configuration Options

### `:throw?` - Exception Behavior

Controls whether spec failures throw exceptions or just log:

```clojure
;; Throw on validation failures (recommended for dev)
{:throw? true}

;; Only log failures, don't throw
{:throw? false}
```

**Default**: `false` (logs but doesn't throw)

**Recommendation**: Set to `true` in development to catch contract violations early.

### `:guardrails/mcps` - Max Checks Per Second

Performance throttling to limit validation overhead:

```clojure
;; Check at most 100 times per second
{:guardrails/mcps 100}

;; No limit (check every call)
{:guardrails/mcps nil}
```

**Default**: `nil` (no throttling)

**Performance Impact**: Without throttling, each check adds ~11 microseconds overhead. With `:mcps 100`, overhead drops to nanoseconds for heavily-called functions (1000x faster).

**Use Case**: Hot paths in development where full checking is too slow.

### `:guardrails/stack-trace` - Error Output Style

Control stack trace verbosity:

```clojure
;; Full stack trace
{:guardrails/stack-trace :full}

;; Pruned stack trace (remove internal frames)
{:guardrails/stack-trace :prune}

;; No stack trace
{:guardrails/stack-trace :none}
```

**Default**: `:prune`

**Options**:
- `:full` - Complete stack trace including Guardrails internals
- `:prune` - Remove internal frames, show only application code
- `:none` - No stack trace, just error message

### `:guardrails/compact?` - Compact Error Messages

Compress error explanations:

```clojure
;; Compact error messages
{:guardrails/compact? true}

;; Verbose error messages
{:guardrails/compact? false}
```

**Default**: `false`

**Effect**: When `true`, reduces the verbosity of validation error messages.

### `:guardrails/trace?` - Call Stack Tracing

Show the instrumented call stack on failures:

```clojure
;; Show call stack
{:guardrails/trace? true}

;; Hide call stack
{:guardrails/trace? false}
```

**Default**: `false`

**Use Case**: Debugging complex call chains to see which functions led to a validation failure.

### `:guardrails/use-stderr?` - Output Target

Control where error messages are printed:

```clojure
;; Print to stderr
{:guardrails/use-stderr? true}

;; Print to stdout
{:guardrails/use-stderr? false}
```

**Default**: `false` (stdout)

**Use Case**: Separate error output from normal logging in production-like environments.

### `:expound` - Error Formatting

Configure Expound error formatting (when using clojure.spec):

```clojure
{:expound {:show-valid-values? true
           :print-specs? true
           :theme :figwheel-theme}}
```

**Options**:
- `:show-valid-values?` - Display valid example values in errors
- `:print-specs?` - Include full spec definitions in errors
- `:theme` - Color theme for error output

## Environment-Based Configuration

### Application Configuration File

Create `resources/config.edn`:

```clojure
{:guardrails
 {:env :dev  ; :dev, :staging, or :production

  ;; Development configuration
  :dev {:enabled true
        :throw-on-error true
        :log-violations true
        :mcps nil}

  ;; Staging configuration
  :staging {:enabled true
            :throw-on-error false
            :log-violations true
            :mcps 100}

  ;; Production configuration
  :production {:enabled false
               :compile-out true}}}
```

### Runtime Configuration

Initialize Guardrails based on environment:

```clojure
(ns myapp.config
  (:require [guardrails.config :as gc]))

(defn init-guardrails! []
  (let [env (or (System/getenv "APP_ENV") "dev")]
    (case env
      "production" (gc/disable-guards!)
      "staging" (do
                  (gc/enable-guards!)
                  (gc/set-mcps! 100))
      "dev" (gc/enable-guards!))))

;; Call at startup
(init-guardrails!)
```

### Environment Variables

```bash
# Set environment
export APP_ENV=production

# Enable/disable at runtime
export GUARDRAILS_ENABLED=true

# Set max checks per second
export GUARDRAILS_MCPS=100
```

## Runtime Control

### Dynamic Enable/Disable

Control guards at runtime without restarts:

```clojure
(require '[guardrails.config :as gc])

;; Enable all guards
(gc/enable-guards!)

;; Disable all guards
(gc/disable-guards!)

;; Check current state
(gc/enabled?)  ;; => true/false
```

### Selective Exclusions

Exclude specific namespaces or functions:

```clojure
;; Exclude entire namespace
(gc/exclude-checks! 'myapp.performance.critical)

;; Exclude specific function
(gc/exclude-checks! 'myapp.core/hot-path-function)

;; Re-enable after exclusion
(gc/allow-checks! 'myapp.performance.critical)
```

### Static Exclusions (Library Authors)

Library authors can mark internal functions to be excluded from downstream validation:

```clojure
;; In library code
(defn ^:guardrails/exclude internal-helper
  [x]
  ;; Will never be checked, even if downstream enables guardrails
  (expensive-operation x))
```

## Guardrails Modes

### Development Mode (`:all`)

Full validation with runtime enforcement:

```bash
clojure -J-Dguardrails.mode=:all -J-Dguardrails.enabled=true
```

**Features**:
- Runtime contract checking
- Full error reporting
- Performance overhead acceptable for development

### Production Mode (`:pro`)

Static analysis only, no runtime overhead:

```bash
clojure -J-Dguardrails.mode=:pro
```

**Features**:
- Static analysis during compilation
- No runtime validation
- Zero performance overhead
- Contracts compiled out of final build

### Disabled Mode

No validation or analysis:

```bash
clojure -J-Dguardrails.enabled=false
```

**Use Case**: Production builds where performance is critical and contracts are not needed.

## Production Configuration

### Compile Out Contracts

#### ClojureScript (shadow-cljs)

```clojure
;; shadow-cljs.edn
{:builds
 {:app
  {:target :browser
   :release
   {:compiler-options
    {:closure-defines {guardrails.config/ENABLED false}}}}}}
```

#### Clojure

```clojure
;; profiles.clj or deps.edn
{:aliases
 {:prod {:jvm-opts ["-Dguardrails.enabled=false"]}}}
```

### Dead Code Elimination

Use noop namespace for maximum optimization:

```clojure
;; In production build
{:closure-defines {guardrails.config/USE_NOOP true}}
```

**Effect**: Replaces all Guardrails functions with no-ops, enabling aggressive dead code elimination by the compiler.

### Verification

Verify guards are disabled in production:

```clojure
(ns myapp.health
  (:require [guardrails.config :as gc]))

(defn health-check []
  {:status "ok"
   :guardrails {:enabled (gc/enabled?)
                :mode (gc/mode)}
   :environment (System/getenv "APP_ENV")})

;; Should return {:guardrails {:enabled false}}
```

## Performance Tuning

### Hot Path Optimization

For heavily-called functions:

```clojure
;; Option 1: Use mcps throttling
(gc/set-mcps! 100)

;; Option 2: Exclude specific functions
(gc/exclude-checks! 'myapp.core/calculate-pixel-color)

;; Option 3: Remove contract entirely for production
#?(:clj
   (>defn hot-function [x]
     [int? => int?]
     (expensive-calculation x))
   :cljs
   (defn hot-function [x]
     (expensive-calculation x)))
```

### Benchmarking

Measure contract overhead:

```clojure
(require '[criterium.core :as crit])

;; With guards enabled
(gc/enable-guards!)
(crit/quick-bench (my-function test-data))

;; With guards disabled
(gc/disable-guards!)
(crit/quick-bench (my-function test-data))

;; With mcps throttling
(gc/enable-guards!)
(gc/set-mcps! 100)
(crit/quick-bench (my-function test-data))
```

### Performance Guidelines

**Overhead without throttling**:
- ~11 microseconds per check on Apple M1
- 100x slower for simple functions

**With `:mcps 100`**:
- Nanosecond overhead (1000x faster)
- Intermittent checks spread evenly

**Recommendations**:
1. Full checking for business logic layers
2. Throttled checking (`:mcps 100`) for service layers
3. Excluded or disabled for hot paths (rendering, pixel operations, etc.)

## Configuration Profiles

### Example: Development Profile

```clojure
;; dev.edn
{:guardrails
 {:enabled true
  :throw? true
  :mcps nil
  :stack-trace :prune
  :trace? true
  :compact? false
  :use-stderr? false
  :expound {:show-valid-values? true
            :print-specs? true}}}
```

### Example: CI/CD Profile

```clojure
;; ci.edn
{:guardrails
 {:enabled true
  :throw? true
  :mcps nil
  :stack-trace :full
  :trace? true
  :compact? false
  :use-stderr? true}}
```

### Example: Staging Profile

```clojure
;; staging.edn
{:guardrails
 {:enabled true
  :throw? false
  :mcps 100
  :stack-trace :none
  :trace? false
  :compact? true
  :use-stderr? true}}
```

### Example: Production Profile

```clojure
;; prod.edn
{:guardrails
 {:enabled false}}
```

## Troubleshooting Configuration

### Guards Not Running

Check:
1. Is `:guardrails.enabled` system property set to `true`?
2. Is the namespace requiring `guardrails.core` correctly?
3. Is there a `guardrails.edn` overriding settings?

```bash
# Verify configuration
clojure -J-Dguardrails.enabled=true -e "(require '[guardrails.config :as gc]) (gc/enabled?)"
```

### Performance Issues in Development

Solutions:
1. Enable `:mcps` throttling
2. Exclude hot paths
3. Use simpler schemas for frequently-called functions

```clojure
;; Add to guardrails.edn
{:guardrails/mcps 100}
```

### Production Contracts Not Removed

Verify:
1. Check closure defines in ClojureScript build
2. Verify JVM opts in Clojure build
3. Use advanced compilation for ClojureScript

```bash
# Check if guards are disabled
java -jar myapp.jar -e "(require '[guardrails.config :as gc]) (gc/enabled?)"
# Should print: false
```

## Best Practices

1. **Always disable in production** - Use environment-based configuration
2. **Use `:throw? true` in dev** - Catch violations early
3. **Throttle in staging** - Balance checking with performance
4. **Verify at startup** - Log Guardrails status in health checks
5. **Profile hot paths** - Exclude or throttle performance-critical code
6. **Version configuration** - Keep configs in source control
7. **Document exclusions** - Comment why specific functions are excluded

## References

- Guardrails GitHub: https://github.com/fulcrologic/guardrails
- Configuration Examples: https://github.com/fulcrologic/guardrails/tree/main/examples
