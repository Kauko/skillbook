# clj-kondo Custom Hooks Reference

Custom hooks enable extending clj-kondo's analysis for macros and special forms it doesn't understand by default. This document covers hook development, API, and best practices.

## Overview

Hooks are Clojure code interpreted using the Small Clojure Interpreter (SCI). They receive code as rewrite-clj nodes and can:
- Transform macro calls into analyzable forms
- Register custom lint findings
- Access namespace analysis data
- Resolve symbols to their definitions

## Hook Types

### Analyze-Call Hooks

Transform or inspect macro/function calls. Provides precise control and location tracking.

**Use when:**
- Need precise error locations
- Require complex transformation logic
- Want to register custom findings

### Macroexpand Hooks

Expand macros using regular Clojure functions. Simpler but with reduced location precision.

**Use when:**
- Transformation is straightforward
- Location precision is less critical
- Want familiar macro syntax

## Directory Structure

Place hook files in `.clj-kondo/` with standard namespace-to-path mapping:

```
.clj-kondo/
├── config.edn
└── hooks/
    ├── my_hooks.clj          ;; hooks.my-hooks
    └── company/
        └── project.clj       ;; hooks.company.project
```

Shared utility code:
```
.clj-kondo/
├── hooks/
│   └── utils.clj             ;; hooks.utils
└── my_app/
    └── analysis.clj          ;; my-app.analysis
```

## Configuration

Register hooks in `.clj-kondo/config.edn`:

```clojure
{:hooks {:analyze-call {my.lib/defcomponent hooks.my-lib/defcomponent
                        my.lib/with-db hooks.my-lib/with-db}
         :macroexpand {my.lib/simple-macro hooks.my-lib/expand-simple}}}
```

## Analyze-Call Hooks API

### Hook Function Signature

```clojure
(ns hooks.my-hooks
  (:require [clj-kondo.hooks-api :as api]))

(defn my-hook
  [{:keys [node config callstack filename]}]
  {:node transformed-node})
```

**Parameters:**
- `:node` - The call expression as rewrite-clj node
- `:config` - Current clj-kondo configuration
- `:callstack` - List of parent calls
- `:filename` - Current file path

**Return:**
- Map with `:node` key containing transformed node

### Node Construction

#### Creating Nodes

```clojure
;; List nodes
(api/list-node [(api/token-node 'let)
                (api/vector-node [])
                (api/token-node 'nil)])
;; => (let [] nil)

;; Vector nodes
(api/vector-node [(api/token-node 'x)
                  (api/token-node 'y)])
;; => [x y]

;; Map nodes
(api/map-node [(api/keyword-node :a)
               (api/token-node 1)])
;; => {:a 1}

;; String nodes
(api/string-node "hello")
;; => "hello"

;; Keyword nodes
(api/keyword-node :foo)
;; => :foo

;; Token nodes (symbols, numbers, etc.)
(api/token-node 'my-symbol)
(api/token-node 42)
```

#### Node Predicates

```clojure
(api/list-node? node)
(api/vector-node? node)
(api/map-node? node)
(api/string-node? node)
(api/keyword-node? node)
(api/token-node? node)
```

### Node Manipulation

#### Accessing Node Properties

```clojure
;; Get child nodes
(:children node)

;; Get node type
(api/tag node)  ;; :list, :vector, :token, etc.

;; Convert to s-expression
(api/sexpr node)  ;; Use sparingly for simple analysis
```

#### Coercing Nodes

```clojure
;; Coerce to different node types
(api/coerce node)  ;; Auto-detect type
(api/vector-node (api/sexpr node))  ;; Manual coercion
```

### Symbol Resolution

```clojure
;; Resolve symbol to namespace-qualified form
(api/resolve {:node (api/token-node 'str/join)})
;; => clojure.string/join

;; Check if symbol is from specific namespace
(let [resolved (api/resolve {:node symbol-node})]
  (= 'clojure.core/defn resolved))
```

### Namespace Analysis

```clojure
;; Get cached analysis for namespace
(api/ns-analysis 'clojure.string)
;; => {:vars {join {:fixed-arities #{2}
;;                  :name 'clojure.string/join
;;                  :ns 'clojure.string}
;;            ...}}
```

### Registering Findings

```clojure
(api/reg-finding! {:type :custom-warning
                   :message "Custom lint message"
                   :row (:row (meta node))
                   :col (:col (meta node))
                   :end-row (:end-row (meta node))
                   :end-col (:end-col (meta node))})
```

Configure custom finding level:
```clojure
{:linters {:custom-warning {:level :warning}}}
```

### Accessing Callstack

```clojure
(defn my-hook [{:keys [callstack]}]
  (let [parent (first callstack)]
    (when (= 'my.ns/parent-macro (:name parent))
      ;; Special handling when called from parent-macro
      ...)))
```

## Analyze-Call Hook Examples

### Example 1: Simple Binding Form

Transform custom binding form to `let`:

```clojure
;; Macro definition
(defmacro with-bound [bindings & body]
  `(let ~bindings ~@body))

;; Hook
(ns hooks.with-bound
  (:require [clj-kondo.hooks-api :as api]))

(defn with-bound [{:keys [node]}]
  (let [[_macro-name bindings & body] (:children node)
        new-node (api/list-node
                  (list*
                   (api/token-node 'let)
                   bindings
                   body))]
    {:node new-node}))

;; Configuration
{:hooks {:analyze-call {my.lib/with-bound hooks.with-bound/with-bound}}}
```

### Example 2: Def-Like Form

Transform custom def form:

```clojure
;; Macro: (defcomponent my-comp [props] ...)
;; Transform to: (def my-comp (fn [props] ...))

(ns hooks.defcomponent
  (:require [clj-kondo.hooks-api :as api]))

(defn defcomponent [{:keys [node]}]
  (let [[_macro name args & body] (:children node)
        new-node (api/list-node
                  [(api/token-node 'def)
                   name
                   (api/list-node
                    (list*
                     (api/token-node 'fn)
                     args
                     body))])]
    {:node new-node}))
```

### Example 3: Custom Validation

Register findings for misuse:

```clojure
(ns hooks.validate-query
  (:require [clj-kondo.hooks-api :as api]))

(defn validate-query [{:keys [node]}]
  (let [[_macro query & _] (:children node)
        query-sexpr (api/sexpr query)]
    ;; Validate query structure
    (when-not (and (vector? query-sexpr)
                   (keyword? (first query-sexpr)))
      (api/reg-finding! {:type :invalid-query
                        :message "Query must be vector starting with keyword"
                        :row (:row (meta query))
                        :col (:col (meta query))
                        :end-row (:end-row (meta query))
                        :end-col (:end-col (meta query))}))
    ;; Return node unchanged
    {:node node}))

;; Configuration
{:linters {:invalid-query {:level :error}}
 :hooks {:analyze-call {my.db/query hooks.validate-query/validate-query}}}
```

### Example 4: Metadata Preservation

Apply metadata to suppress specific linters:

```clojure
(ns hooks.ignore-unused
  (:require [clj-kondo.hooks-api :as api]))

(defn with-cleanup [{:keys [node]}]
  (let [[_macro resource & body] (:children node)
        ;; Ignore unused-binding for cleanup vars
        new-resource (with-meta resource
                       {:clj-kondo/ignore [:unused-binding]})
        new-node (api/list-node
                  (list*
                   (api/token-node 'let)
                   (api/vector-node [new-resource (api/token-node nil)])
                   body))]
    {:node new-node}))
```

## Macroexpand Hooks API

### Hook Function Signature

```clojure
(ns hooks.my-expand
  (:require [clj-kondo.hooks-api :as api]))

(defn expand-my-macro [{:keys [node]}]
  (let [[_macro & args] (api/sexpr node)]
    ;; Return expanded s-expression
    `(let [x# ~(first args)]
       ~@(rest args))))

;; Configuration
{:hooks {:macroexpand {my.lib/my-macro hooks.my-expand/expand-my-macro}}}
```

### Macroexpand Example

```clojure
(ns hooks.when-let-expand
  (:require [clj-kondo.hooks-api :as api]))

(defn when-let [{:keys [node]}]
  (let [[_macro [binding test] & body] (api/sexpr node)]
    `(let [~binding ~test]
       (when ~binding
         ~@body))))

;; Configuration
{:hooks {:macroexpand {my.lib/when-let hooks.when-let-expand/when-let}}}
```

## Development Workflow

### REPL Development

```clojure
;; Start REPL with clj-kondo on classpath
(require '[clj-kondo.hooks-api :as api])

;; Parse test code
(def test-node (api/parse-string "(my-macro [x 1] (inc x))"))

;; Load and test hook
(load-file ".clj-kondo/hooks/my_hooks.clj")
(require '[hooks.my-hooks :as h])

;; Test transformation
(h/my-hook {:node test-node})

;; Enable hot reload
(set! api/*reload* true)
```

### Debugging

```clojure
(ns hooks.debug
  (:require [clj-kondo.hooks-api :as api]))

(defn my-hook [{:keys [node]}]
  ;; Print node structure
  (prn :node node)
  (prn :children (:children node))
  (prn :sexpr (api/sexpr node))

  ;; Print with pretty formatting
  (clojure.pprint/pprint (api/sexpr node))

  {:node node})
```

### Testing Hooks

Create test file:
```clojure
;; test/hooks_test.clj
(ns hooks-test
  (:require [my.lib :refer [my-macro]]))

(my-macro [x 1]
  (inc x))  ;; Test various cases
```

Run clj-kondo:
```bash
clj-kondo --lint test/hooks_test.clj
```

## Performance Optimization

### 1. Minimize Hook Files

Split hooks by usage pattern:
```
.clj-kondo/
└── hooks/
    ├── common.clj      ;; Frequently used
    ├── advanced.clj    ;; Rarely used
    └── utils.clj       ;; Shared utilities
```

### 2. Use Shared Utilities

```clojure
(ns hooks.utils
  (:require [clj-kondo.hooks-api :as api]))

(defn extract-bindings [node]
  (let [[_ bindings] (:children node)]
    bindings))

;; Use in multiple hooks
(ns hooks.my-hook
  (:require [hooks.utils :as utils]))
```

### 3. Avoid Heavy Computation

```clojure
;; BAD: Complex analysis in hook
(defn slow-hook [{:keys [node]}]
  (let [result (expensive-analysis (api/sexpr node))]
    ...))

;; GOOD: Simple transformation
(defn fast-hook [{:keys [node]}]
  (let [[macro & args] (:children node)]
    {:node (api/list-node (list* (api/token-node 'do) args))}))
```

## Common Patterns

### Pattern 1: Let-Like Forms

```clojure
(defn transform-to-let [{:keys [node]}]
  (let [[_macro bindings & body] (:children node)]
    {:node (api/list-node
            (list*
             (api/token-node 'let)
             bindings
             body))}))
```

### Pattern 2: Def-Like Forms

```clojure
(defn transform-to-def [{:keys [node]}]
  (let [[_macro name & rest] (:children node)]
    {:node (api/list-node
            (list*
             (api/token-node 'def)
             name
             rest))}))
```

### Pattern 3: Do-Like Forms

```clojure
(defn transform-to-do [{:keys [node]}]
  (let [[_macro & body] (:children node)]
    {:node (api/list-node
            (list*
             (api/token-node 'do)
             body))}))
```

### Pattern 4: Conditional Transformation

```clojure
(defn conditional-transform [{:keys [node config]}]
  (let [[_macro & args] (:children node)
        strict-mode? (get-in config [:linters :my-linter :strict])]
    (if strict-mode?
      {:node (strict-transform args)}
      {:node (lenient-transform args)})))
```

## Error Handling

### Validation with Findings

```clojure
(defn validate-hook [{:keys [node]}]
  (let [[_macro & args] (:children node)]
    (when (< (count args) 2)
      (api/reg-finding! {:type :invalid-macro-usage
                        :message "Macro requires at least 2 arguments"
                        :row (:row (meta node))
                        :col (:col (meta node))}))
    {:node node}))
```

### Graceful Degradation

```clojure
(defn safe-hook [{:keys [node]}]
  (try
    (let [transformed (complex-transformation node)]
      {:node transformed})
    (catch Exception e
      ;; Fall back to original node
      (api/reg-finding! {:type :hook-error
                        :message (str "Hook error: " (.getMessage e))
                        :row (:row (meta node))
                        :col (:col (meta node))})
      {:node node})))
```

## Best Practices

1. **Keep hooks simple**: Transform to well-known forms (let, def, fn, do)
2. **Preserve metadata**: Location info enables precise error reporting
3. **Test thoroughly**: Create test files covering edge cases
4. **Document behavior**: Add comments explaining transformations
5. **Use sexpr sparingly**: Work with nodes when possible for performance
6. **Handle errors gracefully**: Don't crash, register findings instead
7. **Share common code**: Extract utilities to reduce duplication
8. **Consider maintenance**: Simpler hooks are easier to maintain

## Advanced Topics

### Accessing Parent Context

```clojure
(defn context-aware-hook [{:keys [callstack]}]
  (let [parent (first callstack)
        in-defn? (= 'clojure.core/defn (:name parent))]
    (when in-defn?
      ;; Special handling inside defn
      ...)))
```

### Multi-Arity Functions

```clojure
(defn multi-arity-hook [{:keys [node]}]
  (let [[_macro name & bodies] (:children node)
        ;; Each body is an arity
        fn-node (api/list-node
                 (list*
                  (api/token-node 'fn)
                  name
                  bodies))]
    {:node (api/list-node
            [(api/token-node 'def)
             name
             fn-node])}))
```

### Dynamic Configuration

```clojure
(defn config-driven-hook [{:keys [node config]}]
  (let [opts (get-in config [:linters :my-macro :options])
        [_macro & args] (:children node)]
    {:node (transform-with-options args opts)}))
```

## Troubleshooting

### Hook Not Running

1. Check file location matches namespace
2. Verify configuration syntax
3. Check for syntax errors in hook file
4. Enable debug output: `clj-kondo --debug`

### Incorrect Transformations

1. Print node structure: `(prn node)`
2. Check sexpr output: `(prn (api/sexpr node))`
3. Verify metadata preservation
4. Test in REPL with sample inputs

### Performance Issues

1. Profile hook execution time
2. Minimize sexpr usage
3. Split into multiple hook files
4. Cache expensive computations

## References

- [Official Hooks Documentation](https://github.com/clj-kondo/clj-kondo/blob/master/doc/hooks.md)
- [Hook Examples](https://github.com/clj-kondo/clj-kondo/tree/master/examples)
- [rewrite-clj Documentation](https://github.com/clj-commons/rewrite-clj)
- [Configuration Guide](./configuration.md)
