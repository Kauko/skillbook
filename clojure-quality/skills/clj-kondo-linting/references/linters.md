# clj-kondo Linters Reference

This document catalogs 100+ linters available in clj-kondo. For workflow guidance and common fixes, see the main skill file.

## Linter Categories

### Namespace & Require Management

#### `:aliased-namespace-symbol`
**Default:** `:off`
**Detects:** Using qualified symbols when an alias exists

```clojure
;; WARNING
(ns my.ns (:require [clojure.string :as str]))
(clojure.string/join "," ["a" "b"])  ;; Should use str/join

;; GOOD
(str/join "," ["a" "b"])
```

#### `:conflicting-alias`
**Default:** `:error`
**Detects:** Duplicate alias names

```clojure
;; ERROR
(ns my.ns
  (:require [foo.bar :as util]
            [baz.qux :as util]))  ;; Conflicting alias
```

#### `:consistent-alias`
**Default:** `:warning`
**Detects:** Inconsistent namespace aliases across project

Configuration:
```clojure
{:linters {:consistent-alias {:aliases {clojure.string str
                                        clojure.set set}}}}
```

#### `:unused-namespace`
**Default:** `:warning`
**Detects:** Required but unused namespaces

```clojure
;; WARNING
(ns my.ns
  (:require [clojure.set :as set]))  ;; Never used

;; Configuration
{:linters {:unused-namespace {:exclude [clojure.test]}}}
```

#### `:duplicate-require`
**Default:** `:warning`
**Detects:** Same namespace required multiple times

#### `:self-requiring-namespace`
**Default:** `:warning`
**Detects:** Namespace requiring itself

#### `:unsorted-required-namespaces`
**Default:** `:off`
**Detects:** Unsorted require clauses

### Variable Definition & Usage

#### `:redefined-var`
**Default:** `:warning`
**Detects:** Redefining existing vars

```clojure
;; WARNING
(def x 1)
(def x 2)  ;; Redefinition
```

#### `:unused-private-var`
**Default:** `:warning`
**Detects:** Unused private definitions

```clojure
;; WARNING
(defn- helper [x] (inc x))  ;; Never called
```

#### `:unused-referred-var`
**Default:** `:warning`
**Detects:** Unused `:refer` imports

```clojure
;; WARNING
(ns my.ns
  (:require [clojure.string :refer [join split]]))
(join "," ["a" "b"])  ;; split is unused
```

#### `:unused-binding`
**Default:** `:warning`
**Detects:** Unused let bindings

```clojure
;; WARNING
(let [x 1
      y 2]  ;; y is unused
  x)

;; Configuration
{:linters {:unused-binding {:exclude-destructured-as true}}}
```

#### `:shadowed-var`
**Default:** `:off`
**Detects:** Local bindings shadowing outer vars

```clojure
;; WARNING (if enabled)
(let [name "foo"]  ;; Shadows clojure.core/name
  name)

;; Configuration
{:linters {:shadowed-var {:level :warning
                          :exclude [name str]}}}
```

#### `:var-same-name-except-case`
**Default:** `:warning`
**Detects:** Vars differing only in case (problematic on case-insensitive filesystems)

### Function & Arity

#### `:invalid-arity`
**Default:** `:error`
**Detects:** Wrong number of arguments

```clojure
;; ERROR
(str/split "hello")  ;; Expects 2 args

;; Configuration
{:linters {:invalid-arity {:skip-args [my.ns/weird-fn]}}}
```

#### `:conflicting-fn-arity`
**Default:** `:error`
**Detects:** Duplicate arities in function definitions

```clojure
;; ERROR
(defn foo
  ([x] x)
  ([x] (* x 2)))  ;; Duplicate 1-arg arity
```

#### `:protocol-method-varargs`
**Default:** `:error`
**Detects:** Varargs in protocol method definitions

#### `:missing-body-in-when`
**Default:** `:warning`
**Detects:** Empty `when` expressions

### Type & Comparison

#### `:type-mismatch`
**Default:** `:error`
**Detects:** Type-incompatible operations

```clojure
;; ERROR
(inc :foo)          ;; Can't inc keyword
(conj "string" 1)   ;; Can't conj to string
```

#### `:equals-nil` / `:equals-true` / `:equals-false`
**Default:** `:off`
**Detects:** Using `=` instead of predicates

```clojure
;; WARNING (if enabled)
(= nil x)    ;; Use (nil? x)
(= true x)   ;; Use (true? x)
(= false x)  ;; Use (false? x)
```

#### `:equals-float`
**Default:** `:off`
**Detects:** Floating-point equality comparisons

```clojure
;; WARNING (if enabled)
(= 0.1 x)  ;; Risky due to floating-point precision
```

#### `:single-operand-comparison`
**Default:** `:warning`
**Detects:** Comparison with single operand

```clojure
;; WARNING
(< 1)  ;; Always true, likely a bug
```

### Control Flow

#### `:missing-else-branch`
**Default:** `:warning`
**Detects:** `if` without else clause

```clojure
;; WARNING
(if condition
  (do-something))  ;; Returns nil if condition is false
```

#### `:cond-else`
**Default:** `:warning`
**Detects:** Non-`:else` catch-all in `cond`

```clojure
;; WARNING
(cond
  (= x 1) "one"
  true "other")  ;; Should be :else

;; GOOD
(cond
  (= x 1) "one"
  :else "other")
```

#### `:loop-without-recur`
**Default:** `:warning`
**Detects:** Loops that never recur

```clojure
;; WARNING
(loop [x 0]
  (println x))  ;; Runs once, then exits
```

#### `:unreachable-code`
**Default:** `:warning`
**Detects:** Code after terminal expressions

```clojure
;; WARNING
(cond
  :else (println "always")
  (= x 1) (println "never"))  ;; Unreachable
```

#### `:unexpected-recur`
**Default:** `:error`
**Detects:** `recur` outside loop/fn context

### Redundancy Detection

#### `:redundant-do`
**Default:** `:warning`
**Detects:** Unnecessary `do` blocks

```clojure
;; WARNING
(if condition
  (do (println "yes"))  ;; Single expression, do not needed
  false)
```

#### `:redundant-let`
**Default:** `:warning`
**Detects:** Nested let bindings

```clojure
;; WARNING
(let [x 1]
  (let [y 2]  ;; Can merge with outer let
    (+ x y)))
```

#### `:redundant-call`
**Default:** `:off`
**Detects:** Unnecessary function applications

```clojure
;; WARNING (if enabled)
(-> x)     ;; Just use x
(doto x)   ;; Just use x
```

#### `:redundant-fn-wrapper`
**Default:** `:off`
**Detects:** Unnecessary lambda wrapping

```clojure
;; WARNING (if enabled)
(map #(inc %) xs)  ;; Just use: (map inc xs)
```

#### `:redundant-nested-call`
**Default:** `:info`
**Detects:** Nested identical operations

```clojure
;; INFO
(+ (+ 1 2) 3)  ;; Could be (+ 1 2 3)
(str (str "a" "b") "c")  ;; Could be (str "a" "b" "c")
```

#### `:redundant-str-call`
**Default:** `:off`
**Detects:** Unnecessary `str` calls

```clojure
;; WARNING (if enabled)
(str "literal")  ;; Just use "literal"
```

### Code Quality

#### `:missing-docstring`
**Default:** `:off`
**Detects:** Public vars without docstrings

```clojure
;; Configuration
{:linters {:missing-docstring {:level :warning
                               :exclude-cljs true}}}
```

#### `:docstring-blank`
**Default:** `:warning`
**Detects:** Empty docstrings

#### `:docstring-no-summary`
**Default:** `:off`
**Detects:** Docstrings not starting with sentence case

#### `:docstring-leading-trailing-whitespace`
**Default:** `:off`
**Detects:** Whitespace issues in docstrings

#### `:misplaced-docstring`
**Default:** `:warning`
**Detects:** Docstrings after parameter vector

```clojure
;; WARNING
(defn foo [x]
  "Does something"  ;; Should be before [x]
  (* x 2))
```

#### `:missing-test-assertion`
**Default:** `:warning`
**Detects:** Test functions without assertions

```clojure
;; WARNING
(deftest my-test
  (println "running"))  ;; No assertions
```

#### `:inline-def`
**Default:** `:warning`
**Detects:** `def` inside function bodies

```clojure
;; WARNING
(defn foo []
  (def x 1)  ;; Should be let
  x)
```

### Collection Issues

#### `:duplicate-map-key`
**Default:** `:error`
**Detects:** Duplicate keys in map literals

```clojure
;; ERROR
{:a 1 :a 2}  ;; Duplicate :a
```

#### `:duplicate-set-element`
**Default:** `:error`
**Detects:** Duplicate elements in set literals

```clojure
;; ERROR
#{1 2 1}  ;; Duplicate 1
```

#### `:missing-map-value`
**Default:** `:error`
**Detects:** Odd number of forms in map literal

```clojure
;; ERROR
{:a 1 :b}  ;; Missing value for :b
```

#### `:not-empty?`
**Default:** `:warning`
**Detects:** `(not (empty? ...))` pattern

```clojure
;; WARNING
(if (not (empty? coll))
  ...)

;; GOOD: Use seq
(when (seq coll)
  ...)
```

#### `:single-key-in`
**Default:** `:off`
**Detects:** `get-in`/`assoc-in` with single key

```clojure
;; WARNING (if enabled)
(get-in m [:key])  ;; Use (get m :key)
```

### Java Interop

#### `:java-static-field-call`
**Default:** `:warning`
**Detects:** Static field access with parens

```clojure
;; WARNING
(System/out)  ;; Field, not method

;; GOOD
System/out
```

#### `:discouraged-java-method`
**Default:** `:warning`
**Detects:** Configurable Java method warnings

```clojure
{:linters {:discouraged-java-method
           {:methods {java.lang.System/exit {:level :error
                                             :message "Don't call System.exit"}}}}}
```

#### `:warn-on-reflection`
**Default:** `:off`
**Detects:** Reflection in Java interop

Enable to catch performance issues:
```clojure
{:linters {:warn-on-reflection {:level :warning}}}
```

### Namespace Unqualified Warnings

#### `:unresolved-symbol`
**Default:** `:error`
**Detects:** Undefined symbols

```clojure
;; Configuration
{:linters {:unresolved-symbol {:exclude [(clojure.core/defn)
                                         (mylib/custom-macro)]
                               :exclude-patterns ["^js/.*"]}}}
```

#### `:unresolved-namespace`
**Default:** `:error`
**Detects:** Undefined namespaces

#### `:unresolved-var`
**Default:** `:error`
**Detects:** Undefined vars in qualified symbols

### Deprecation

#### `:deprecated-var`
**Default:** `:warning`
**Detects:** Usage of deprecated vars

Mark vars as deprecated:
```clojure
(defn old-fn
  {:deprecated "1.2.0"}
  [x]
  x)
```

### Style & Idioms

#### `:refer-all`
**Default:** `:warning`
**Detects:** `:refer :all` usage

```clojure
;; WARNING
(ns my.ns
  (:require [clojure.string :refer :all]))

;; Configuration: allow in tests
{:linters {:refer-all {:exclude [clojure.test]}}}
```

#### `:use`
**Default:** `:warning`
**Detects:** `use` instead of `require`

#### `:unused-import`
**Default:** `:warning`
**Detects:** Unused Java imports

#### `:import-duplicate`
**Default:** `:error`
**Detects:** Duplicate Java imports

### Testing

#### `:multiple-async-in-deftest`
**Default:** `:warning`
**Detects:** Multiple `async` blocks in single test

#### `:deftest-without-testing`
**Default:** `:off`
**Detects:** `deftest` without `testing` blocks

### Special Forms

#### `:case-duplicate-test`
**Default:** `:error`
**Detects:** Duplicate test values in `case`

```clojure
;; ERROR
(case x
  1 "one"
  1 "uno")  ;; Duplicate 1
```

#### `:case-quoted-test`
**Default:** `:warning`
**Detects:** Quoted symbols as case tests

#### `:case-symbol-test`
**Default:** `:off`
**Detects:** Unquoted symbols as case tests

### Syntax & Structure

#### `:syntax`
**Default:** `:error`
**Detects:** Invalid Clojure syntax

#### `:invalid-token`
**Default:** `:error`
**Detects:** Invalid tokens

#### `:reader-tag-syntax`
**Default:** `:error`
**Detects:** Invalid reader tag usage

### ClojureScript Specific

#### `:invalid-symbol-name`
**Default:** `:error` (ClojureScript only)
**Detects:** Invalid symbol names for JS interop

#### `:invalid-protocol-implementation`
**Default:** `:error`
**Detects:** Invalid protocol implementations

### Datalog (Datomic/Datascript)

#### `:datalog-syntax`
**Default:** `:error`
**Detects:** Invalid Datalog query syntax

### Babashka Specific

#### `:bb.edn-*`
Multiple linters for Babashka configuration validation:
- `:bb.edn-undefined-task`
- `:bb.edn-duplicate-task`
- `:bb.edn-task-cyclic-dependency`

### Configuration File Linters

#### `:deps.edn`
**Default:** `:warning`
**Detects:** Issues in `deps.edn` files

## Linter Configuration Patterns

### Severity Levels

```clojure
{:linters {:linter-name {:level :off}}}      ;; Disable
{:linters {:linter-name {:level :info}}}     ;; Informational
{:linters {:linter-name {:level :warning}}}  ;; Warning (default)
{:linters {:linter-name {:level :error}}}    ;; Error
```

### Exclusions

```clojure
{:linters {:linter-name {:exclude [namespace/var
                                   another.ns/fn]
                         :exclude-patterns ["^js/.*" ".*-test$"]}}}
```

### Namespace Filtering

```clojure
{:linters {:linter-name {:namespaces ["^my-app\\..*"]}}}
```

### Arity Restrictions

```clojure
{:linters {:invalid-arity {:skip-args [my.ns/macro]}}}
```

### Language-Specific

```clojure
{:linters {:linter-name {:langs [:clj]}}}      ;; Clojure only
{:linters {:linter-name {:langs [:cljs]}}}     ;; ClojureScript only
{:linters {:linter-name {:langs [:clj :cljs]}}} ;; Both
```

## References

For complete linter documentation, see:
- [Official Linters Documentation](https://github.com/clj-kondo/clj-kondo/blob/master/doc/linters.md)
- [Configuration Guide](./configuration.md)
