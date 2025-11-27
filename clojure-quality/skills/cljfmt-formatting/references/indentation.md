# cljfmt Indentation Rules Reference

This document provides comprehensive reference for configuring custom indentation rules in cljfmt.

## Overview

cljfmt formats code according to the Clojure Style Guide by default. For macros and functions that require different indentation, custom rules can be defined in the configuration.

## Configuration Keys

Two main configuration keys control indentation:

### `:indents`
- Completely replaces ALL default indentation rules
- Use when you want full control over all indentation
- Warning: You lose all built-in rules for clojure.core, compojure, etc.

```clojure
{:indents {when [[:block 1]]
           defn [[:inner 0]]}}
```

### `:extra-indents` (Recommended)
- Adds custom rules while preserving all defaults
- Recommended for most use cases
- Your rules take precedence over defaults when there's overlap

```clojure
{:extra-indents {myapp.core/with-context [[:inner 1]]
                 myapp.routes/defapi [[:block 0]]}}
```

## Pattern Matching

Indent rules support multiple matching strategies:

### 1. Unqualified Symbols
Match regardless of namespace:

```clojure
{:extra-indents {when [[:block 1]]}}
```

Matches:
- `(when test body)`
- `(myns/when test body)`
- `(other.ns/when test body)`

### 2. Qualified Symbols
Match only specific namespace:

```clojure
{:extra-indents {com.example/foo [[:inner 0]]}}
```

Matches only:
- `(com.example/foo args)`

Does not match:
- `(foo args)`
- `(other.ns/foo args)`

### 3. Regular Expressions
Match multiple symbols by pattern:

```clojure
;; In .clj files
{:extra-indents {#"^with-" [[:inner 1]]}}

;; In .edn files (use #re data reader)
{:extra-indents {#re "^with-" [[:inner 1]]}}
```

Matches:
- `(with-open [f file] body)`
- `(with-context [ctx] body)`
- `(with-anything [x] body)`

Common patterns:
```clojure
{:extra-indents {#"^def.*" [[:inner 0]]      ; defn, defmacro, defprotocol, etc.
                 #"^with-.*" [[:inner 1]]     ; with-open, with-redefs, etc.
                 #".*-for-all$" [[:block 1]]  ; prop/for-all, gen/for-all, etc.
                 #"^go-.*" [[:inner 0]]}}     ; go-loop, go-retry, etc.
```

### 4. Vector Notation (Namespace + Pattern)
Combine namespace matching with pattern matching:

```clojure
{:extra-indents {[com.example #"^foo"] [[:inner 0]]}}
```

Matches:
- `(com.example/foo-bar args)`
- `(com.example/foo-baz args)`

Does not match:
- `(other.ns/foo-bar args)`
- `(com.example/bar-foo args)`

## Namespace Aliases

Use `:alias-map` to resolve namespace aliases in indent rules:

```clojure
{:alias-map {test clojure.test
             async clojure.core.async
             s schema.core}

 :extra-indents {test/deftest [[:inner 0]]
                 async/go-loop [[:block 1]]
                 s/defn [[:inner 0]]}}
```

Without `:alias-map`, these rules would need full namespace qualification:
```clojure
{:extra-indents {clojure.test/deftest [[:inner 0]]
                 clojure.core.async/go-loop [[:block 1]]
                 schema.core/defn [[:inner 0]]}}
```

## Three Indentation Rule Types

### 1. Default (No Rule)

When no custom rule is specified, behavior depends on first-line element count:

**One or fewer elements on first line:** Indent by one space
```clojure
(some-function
 arg1
 arg2
 arg3)
```

**Multiple elements on first line:** Align to second element
```clojure
(some-function arg1
               arg2
               arg3)
```

### 2. Inner Indentation

Format: `[:inner depth]` or `[:inner depth index]`

Consistently indents all lines after the first by two spaces, regardless of element count on first line.

#### Basic Inner Rule

```clojure
{:extra-indents {defn [[:inner 0]]}}
```

Result:
```clojure
(defn greet
  "Greet a person"
  [name]
  (println "Hello" name))
```

All lines after the first are indented by 2 spaces, no matter what's on the first line.

#### Understanding Depth

The `depth` parameter specifies which nesting level this rule applies to:

**Depth 0:** Direct children of the form
```clojure
(defn foo [x]       ; rule: [[:inner 0]]
  (+ x 1))          ; <- depth 0 (direct child)
```

**Depth 1:** Grandchildren (nested one level deeper)
```clojure
(proxy [Interface] []   ; rule: [[:inner 0] [:inner 1]]
  (method1 []             ; <- depth 0
    body))                ; <- depth 1
```

**Multiple depths:**
```clojure
{:extra-indents {proxy [[:inner 0] [:inner 1]]}}
```

#### Index Restriction

Optional index parameter restricts which argument position the rule applies to:

```clojure
{:extra-indents {fdef [[:block 1] [:inner 2 0]]}}
```

The `[:inner 2 0]` means: "Apply inner indentation at depth 2, but only for argument at index 0"

### 3. Block Indentation

Format: `[:block index]`

Hybrid approach: uses default rules up to a specified argument index, then switches to inner-style indentation if that argument starts a new line.

#### Basic Block Rule

```clojure
{:extra-indents {when [[:block 0]]}}
```

If argument 0 (first argument) is NOT at line start, use default indentation:
```clojure
(when condition (do-something)
                (do-another))
```

If argument 0 IS at line start, use inner indentation:
```clojure
(when
  condition
  (do-something)
  (do-another))
```

More commonly:
```clojure
(when condition
  (do-something)
  (do-another))
```

#### Block with Higher Index

```clojure
{:extra-indents {let [[:block 1]]}}
```

First argument (index 0, the bindings vector) can be on same line. Starting from index 1 (the body), if it's on new line, use inner indentation:

```clojure
(let [x 1
      y 2]
  (+ x y))
```

Another example with `defn`:
```clojure
{:extra-indents {defn [[:block 2]]}}
```

```clojure
(defn greet
  [name]
  (println "Hello" name))
```

Arguments 0-1 (name, parameters) can use default rules. From argument 2 onward (body), use inner indentation.

## Core Concepts

### Index
Argument position starting from zero. The second element in a form has index 0.

```clojure
(foo arg0 arg1 arg2)
     ^    ^    ^
   index: 0    1    2
```

### Depth
Nesting level relative to the parent form. Direct children are depth 0.

```clojure
(parent
  child-depth-0
  (nested
    child-depth-1))
```

Indentation rules apply to the first element at each depth on every line.

## Common Macro Patterns

### Macros with Body

Body starts immediately (no bindings or special forms first):

```clojure
;; Macro definition
(defmacro with-timing [& body]
  `(time (do ~@body)))

;; Indent rule
{:extra-indents {myapp.macros/with-timing [[:block 0]]}}

;; Formatted result
(with-timing
  (do-something)
  (do-another))
```

### Macros with Bindings and Body

First argument is bindings, rest is body:

```clojure
;; Macro definition
(defmacro with-context [bindings & body]
  ...)

;; Indent rule
{:extra-indents {myapp.macros/with-context [[:block 1]]}}

;; Formatted result
(with-context [ctx (get-context)]
  (process ctx)
  (cleanup ctx))
```

### Macros with Multiple Special Forms

Multiple levels of nesting with different indentation:

```clojure
;; Macro definition
(defmacro defapi [name routes & handlers]
  ...)

;; Indent rule
{:extra-indents {myapp.routes/defapi [[:block 0]]}}

;; Formatted result
(defapi my-api
  "/api/v1"
  (GET "/users" [] (get-users))
  (POST "/users" [] (create-user)))
```

### Testing Macros

```clojure
{:extra-indents {clojure.test/deftest [[:inner 0]]
                 clojure.test/testing [[:inner 0]]
                 clojure.test/use-fixtures [[:inner 0]]}}
```

Result:
```clojure
(deftest my-test
  (testing "something"
    (is (= 1 1))))
```

### Specification Macros

```clojure
{:extra-indents {clojure.spec.alpha/fdef [[:block 1]]}}
```

Result:
```clojure
(s/fdef my-function
  :args (s/cat :x number?)
  :ret number?)
```

## Complete Real-World Example

Here's a comprehensive configuration for a typical Clojure project:

```clojure
{:extra-indents {;; Core patterns
                 #"^def.*" [[:inner 0]]
                 #"^with-.*" [[:inner 1]]
                 #"^let.*" [[:block 1]]

                 ;; Core macros requiring custom rules
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

                 ;; Component libraries
                 integrant.core/defmethod-ig [[:inner 0]]
                 component/start [[:inner 0]]
                 component/stop [[:inner 0]]
                 mount.core/defstate [[:inner 0]]

                 ;; Guardrails
                 guardrails.core/defn [[:inner 0]]
                 guardrails.core/>defn [[:inner 0]]
                 guardrails.core/>defn- [[:inner 0]]

                 ;; Property testing
                 #".*-for-all$" [[:block 1]]}

 :alias-map {test clojure.test
             s schema.core
             m malli.core
             async clojure.core.async
             rf re-frame.core}}
```

## Advanced Techniques

### Multiple Rules per Symbol

Apply different rules at different depths:

```clojure
{:extra-indents {proxy [[:inner 0] [:inner 1]]}}
```

This ensures both interface methods AND their bodies are indented consistently.

### Conditional Indentation with Index

Apply rules only to specific arguments:

```clojure
{:extra-indents {fdef [[:block 1] [:inner 2 0]]}}
```

The `[:inner 2 0]` applies inner indentation at depth 2, but only for the first argument (index 0).

### Regex Pattern Families

Group related macros:

```clojure
{:extra-indents {;; All def* forms
                 #"^def.*" [[:inner 0]]

                 ;; All with-* resource macros
                 #"^with-.*" [[:inner 1]]

                 ;; All testing property generators
                 #".*-for-all$" [[:block 1]]

                 ;; All core.async go-* forms
                 #"^go-.*" [[:inner 0]]}}
```

### Namespace-Specific Overrides

Override default behavior for specific namespaces:

```clojure
{:extra-indents {;; Default 'when' is fine everywhere
                 ;; except in our custom namespace
                 myapp.control/when [[:inner 1]]}}
```

## Troubleshooting Indentation

### Rule Not Applying

1. **Check namespace qualification:** Unqualified rules match all namespaces. Qualified rules match only specific namespace.
   ```clojure
   ;; This matches any/all 'foo'
   {:extra-indents {foo [[:inner 0]]}}

   ;; This matches only 'myapp.core/foo'
   {:extra-indents {myapp.core/foo [[:inner 0]]}}
   ```

2. **Verify alias-map:** If using namespace aliases, ensure they're defined:
   ```clojure
   {:alias-map {test clojure.test}
    :extra-indents {test/deftest [[:inner 0]]}}
   ```

3. **Check regex syntax:** In EDN files, use `#re` data reader:
   ```clojure
   ;; .clj file
   {:extra-indents {#"^def.*" [[:inner 0]]}}

   ;; .edn file
   {:extra-indents {#re "^def.*" [[:inner 0]]}}
   ```

4. **Verify rule format:** Rules must be vectors of vectors:
   ```clojure
   ;; CORRECT
   {:extra-indents {foo [[:inner 0]]}}
   {:extra-indents {foo [[:block 1]]}}
   {:extra-indents {foo [[:inner 0] [:inner 1]]}}

   ;; WRONG
   {:extra-indents {foo [:inner 0]}}
   {:extra-indents {foo :inner}}
   ```

### Unexpected Indentation

1. **Check for conflicting rules:** More specific rules override general ones. Qualified symbols override unqualified:
   ```clojure
   {:extra-indents {foo [[:inner 0]]              ; General rule
                    myapp.core/foo [[:block 1]]}} ; Overrides for myapp.core/foo
   ```

2. **Verify depth parameter:** Ensure depth matches your nesting structure:
   ```clojure
   (parent                ; <- form
     child-depth-0        ; <- depth 0
     (nested              ; <- still depth 0
       child-depth-1))    ; <- depth 1
   ```

3. **Check index restrictions:** If using index parameter, verify argument positions:
   ```clojure
   (form arg0 arg1 arg2)
         ^    ^    ^
      index: 0  1    2
   ```

### Lost Default Rules

If indentation for core Clojure forms stops working:

**Problem:** You used `:indents` instead of `:extra-indents`

**Solution:** Change to `:extra-indents` to preserve defaults:
```clojure
;; WRONG - loses all defaults
{:indents {myapp.core/foo [[:inner 0]]}}

;; CORRECT - keeps defaults
{:extra-indents {myapp.core/foo [[:inner 0]]}}
```

**Temporary workaround:** Use legacy merge mode:
```clojure
{:legacy/merge-indents? true
 :indents {myapp.core/foo [[:inner 0]]}}
```

## Testing Your Rules

To verify custom indentation rules:

1. **Create a test file** with intentionally misformatted code:
   ```clojure
   (myapp.core/with-context [ctx (get-ctx)]
   (process ctx)
   (cleanup ctx))
   ```

2. **Run cljfmt check:**
   ```bash
   clojure -Tcljfmt check
   ```

3. **Review suggested formatting:**
   ```bash
   clojure -Tcljfmt fix
   ```

4. **Verify result matches expectations:**
   ```clojure
   (myapp.core/with-context [ctx (get-ctx)]
     (process ctx)
     (cleanup ctx))
   ```

## Default Indentation Rules

cljfmt includes built-in indentation rules for:

- Clojure core macros and special forms
- Compojure routing macros
- Common macro patterns (def*, with-*, let*)

These rules are automatically applied unless you:
1. Use `:indents` (which replaces all defaults)
2. Override with more specific rule in `:extra-indents`

View source for complete list:
https://github.com/weavejester/cljfmt/blob/master/cljfmt/resources/cljfmt/indents/clojure.clj

## Best Practices

1. **Use `:extra-indents`** instead of `:indents` to preserve defaults
2. **Document custom rules** in comments explaining why they're needed
3. **Test rules** on real code before committing to version control
4. **Use regex patterns** for macro families (def*, with-*, etc.)
5. **Prefer unqualified symbols** unless you need namespace-specific behavior
6. **Define alias-map** for commonly aliased namespaces
7. **Start simple** with `:inner` or `:block`, add complexity only if needed
8. **Commit `.cljfmt.edn`** to version control for team consistency

## Additional Resources

- Main configuration documentation: See `configuration.md` in this directory
- cljfmt repository: https://github.com/weavejester/cljfmt
- Clojure Style Guide: https://guide.clojure.style
- Default indentation rules source: https://github.com/weavejester/cljfmt/blob/master/cljfmt/resources/cljfmt/indents/clojure.clj
