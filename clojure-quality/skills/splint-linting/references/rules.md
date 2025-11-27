# Splint Rules Reference

This document provides comprehensive documentation of all Splint linting rules, organized by category.

## Rule Categories Overview

Splint organizes rules into five main categories:

- **Lint**: Potentially incorrect or suspicious patterns
- **Style**: Clojure community idioms and style conventions
- **Naming**: Identifier naming conventions
- **Performance**: Optimization suggestions (disabled by default)
- **Metrics**: Measurable code properties like function length

**Key Concepts:**
- Rules marked "enabled by default" run automatically unless disabled
- "Safe" rules should not produce false positives
- "Autocorrect" rules can be automatically fixed with `--autocorrect`
- **Warning**: Autocorrect removes all comments and uneval blocks

## Lint Rules

Lint rules address potentially incorrect or suspicious code patterns.

### Control Flow Optimization

#### lint/if-else-nil
Use `when` instead of `if` without an else branch.

```clojure
; Bad
(if (some-func) :a nil)

; Good
(when (some-func) :a)
```

#### lint/if-let-else-nil
Use `when-let` instead of `if-let` without an else branch.

```clojure
; Bad
(if-let [x (some-func)] x nil)

; Good
(when-let [x (some-func)] x)
```

#### lint/if-nil-else
Use `if-some` or `when-some` for nil checks.

```clojure
; Bad
(if (nil? x) nil :value)

; Good
(if-some [_ x] :value)
```

#### lint/if-not-both
Simplify `(if (not x) a b)` to `(if x b a)`.

```clojure
; Bad
(if (not condition) :false-branch :true-branch)

; Good
(if condition :true-branch :false-branch)
```

#### lint/if-not-do
Use `when-not` instead of `(if (not ...) (do ...))`.

```clojure
; Bad
(if (not x) (do (println "no") false))

; Good
(when-not x (println "no") false)
```

#### lint/if-not-not
Remove double negation.

```clojure
; Bad
(if (not (not x)) :truthy :falsy)

; Good
(if x :truthy :falsy)
```

#### lint/if-same-truthy
Both branches return the same value - simplify.

```clojure
; Bad
(if condition :same :same)

; Good
:same
```

#### lint/let-if
Convert `(let [x ...] (if x ...))` to `if-let` or `when-let`.

```clojure
; Bad
(let [x (some-func)]
  (if x (process x)))

; Good
(when-let [x (some-func)]
  (process x))
```

#### lint/let-when
Convert `(let [x ...] (when x ...))` to `when-let`.

```clojure
; Bad
(let [x (some-func)]
  (when x (process x)))

; Good
(when-let [x (some-func)]
  (process x))
```

#### lint/loop-do
Remove unnecessary `do` inside `loop`.

```clojure
; Bad
(loop [i 0]
  (do
    (println i)
    (recur (inc i))))

; Good
(loop [i 0]
  (println i)
  (recur (inc i)))
```

#### lint/loop-empty-when
`loop` with only `when` and `recur` should use `while`.

```clojure
; Bad
(loop []
  (when condition
    (do-work)
    (recur)))

; Good
(while condition
  (do-work))
```

#### lint/missing-body-in-when
`when` without a body always returns nil.

```clojure
; Bad
(when condition)

; Good
(when condition
  :some-value)
```

#### lint/identical-branches
`if` with identical branches can be simplified.

```clojure
; Bad
(if condition
  (println "same")
  (println "same"))

; Good
(println "same")
```

#### lint/no-catch
Avoid empty `catch` blocks.

```clojure
; Bad
(try
  (risky-operation)
  (catch Exception e))

; Good
(try
  (risky-operation)
  (catch Exception e
    (log/error e "Operation failed")))
```

#### lint/thread-macro-one-arg
Threading macro with single argument is unnecessary.

```clojure
; Bad
(-> x)
(->> coll)

; Good
x
coll
```

### Function & Data Transformation

#### lint/assoc-fn
Use `update` instead of `assoc` with function application.

```clojure
; Bad
(assoc coll :a (+ (:a coll) 5))

; Good
(update coll :a + 5)
```

#### lint/dorun-map
Use `run!` instead of `(dorun (map ...))`.

```clojure
; Bad
(dorun (map println (range 10)))

; Good
(run! println (range 10))
```

#### lint/into-literal
Use literal syntax instead of `into`.

```clojure
; Bad
(into [] (range 5))

; Good
(vec (range 5))
```

#### lint/not-empty?
Use `seq` instead of `(not (empty? ...))`.

```clojure
; Bad
(not (empty? coll))

; Good
(seq coll)
```

#### lint/redundant-call
Remove redundant function calls.

```clojure
; Bad
(str "hello")
(vec [1 2 3])

; Good
"hello"
[1 2 3]
```

#### lint/redundant-str-call
Remove unnecessary `str` calls.

```clojure
; Bad
(str "literal")

; Good
"literal"
```

#### lint/take-repeatedly
Use `repeat` or `repeatedly` instead of `(take n (repeat ...))`.

```clojure
; Bad
(take 5 (repeat :x))

; Good
(repeat 5 :x)
```

#### lint/update-with-swap
Use `swap!` directly instead of `update` wrapper.

```clojure
; Bad
(update atom-ref :key inc)

; Good
(swap! atom-ref update :key inc)
```

#### lint/min-max
Simplify single-argument `min` or `max`.

```clojure
; Bad
(min x)
(max x)

; Good
x
x
```

#### lint/divide-by-one
Dividing by 1 is unnecessary.

```clojure
; Bad
(/ x 1)

; Good
x
```

#### lint/rand-int-one
`(rand-int 1)` always returns 0.

```clojure
; Bad
(rand-int 1)

; Good
0
```

#### lint/fn-wrapper
Remove unnecessary function wrapper.

```clojure
; Bad
(fn [x] (inc x))

; Good
inc
```

### Java Interop

#### lint/dot-class-method
Use direct method call syntax instead of dot form.

```clojure
; Bad
(. String valueOf x)

; Good
(String/valueOf x)
```

#### lint/dot-obj-method
Use dot shorthand for object method calls.

```clojure
; Bad
(. obj method args)

; Good
(.method obj args)
```

#### lint/no-target-for-method
Method call missing target object.

```clojure
; Bad
(.toString)

; Good
(.toString obj)
```

#### lint/prefer-method-values
Use method values instead of anonymous function wrappers.

```clojure
; Bad
(fn [x] (.toString x))

; Good
(memfn toString)
```

#### lint/misplaced-type-hint
Type hints should be on the symbol, not the binding.

```clojure
; Bad
(let [^String x "hello"])

; Good
(let [x ^String "hello"])
```

#### lint/require-explicit-param-tags
Functions with varargs need explicit type hints.

#### lint/warn-on-reflection
Add type hints to avoid reflection.

#### lint/catch-throwable
Catching `Throwable` is too broad - catch specific exceptions.

```clojure
; Bad
(catch Throwable t)

; Good
(catch Exception e)
```

### Code Quality & Safety

#### lint/duplicate-case-test
`case` has duplicate test values.

```clojure
; Bad
(case x
  :a "first"
  :a "duplicate")

; Good
(case x
  :a "unique"
  :b "different")
```

#### lint/duplicate-field-name
Record or deftype has duplicate field names.

```clojure
; Bad
(defrecord Person [name name])

; Good
(defrecord Person [name age])
```

#### lint/incorrectly-swapped
Arguments appear to be in wrong order.

#### lint/locking-object
Don't lock on objects that might be reassigned.

#### lint/no-op-assignment
Assignment has no effect.

```clojure
; Bad
(let [x x])

; Good
(let [x (process x)])
```

#### lint/body-unquote-splicing
Unquote-splicing in macro body position may cause issues.

```clojure
; Bad
`(do ~@body)

; Good
`(do ~@(butlast body) ~(last body))
```

#### lint/try-splicing
Splicing in try block may cause incorrect exception handling.

#### lint/existing-constant
Use existing constant instead of magic number.

#### lint/underscore-in-namespace
Avoid underscores in namespace names.

```clojure
; Bad
(ns my_app.core)

; Good
(ns my-app.core)
```

#### lint/defmethod-names
`defmethod` implementations should have descriptive names.

### Module Organization

#### lint/prefer-require-over-use
Use `:require` instead of deprecated `:use`.

```clojure
; Bad
(:use [clojure.string])

; Good
(:require [clojure.string :as str])
```

## Style Rules

Style rules enforce Clojure community idioms and conventions.

### String Operations

#### style/apply-str
Replace `(apply str x)` with `(clojure.string/join x)`.

```clojure
; Bad
(apply str coll)

; Good
(require '[clojure.string :as str])
(str/join coll)
```

#### style/apply-str-interpose
Use `clojure.string/join` instead of `(apply str (interpose...))`.

```clojure
; Bad
(apply str (interpose "," items))

; Good
(str/join "," items)
```

#### style/apply-str-reverse
Prefer `clojure.string/reverse` over `(apply str (reverse...))`.

```clojure
; Bad
(apply str (reverse s))

; Good
(str/reverse s)
```

#### style/reduce-str
Use `clojure.string/join` instead of `reduce` with `str`.

```clojure
; Bad
(reduce str coll)

; Good
(str/join coll)
```

### Comparison Functions

#### style/eq-true
Use `true?` instead of `(= true x)`.

```clojure
; Bad
(= true x)

; Good
(true? x)
```

#### style/eq-false
Use `false?` instead of `(= false x)`.

```clojure
; Bad
(= false x)

; Good
(false? x)
```

#### style/eq-nil
Use `nil?` instead of `(= nil x)`.

```clojure
; Bad
(= nil x)

; Good
(nil? x)
```

#### style/eq-zero
Use `zero?` instead of `(= 0 x)`.

```clojure
; Bad
(= 0 x)

; Good
(zero? x)
```

#### style/pos-checks
Use `pos?` instead of comparison.

```clojure
; Bad
(< 0 num)
(> num 0)

; Good
(pos? num)
```

#### style/neg-checks
Use `neg?` instead of comparison.

```clojure
; Bad
(< num 0)
(> 0 num)

; Good
(neg? num)
```

#### style/not-eq
Use `not=` instead of `(not (= ...))`.

```clojure
; Bad
(not (= x y))

; Good
(not= x y)
```

### Collection Operations

#### style/conj-vector
Prefer `vector` over `(conj [] ...)`.

```clojure
; Bad
(conj [] 1 2 3)

; Good
(vector 1 2 3)
; or
[1 2 3]
```

#### style/filter-vec-filterv
Use `filterv` instead of `(vec (filter ...))`.

```clojure
; Bad
(vec (filter even? coll))

; Good
(filterv even? coll)
```

#### style/filter-complement
Replace `(filter (complement pred) coll)` with `(remove pred coll)`.

```clojure
; Bad
(filter (complement even?) coll)

; Good
(remove even? coll)
```

#### style/single-key-in
Use `assoc` instead of `assoc-in` with a single key.

```clojure
; Bad
(assoc-in m [:key] value)

; Good
(assoc m :key value)
```

### Nested Functions

#### style/first-first
Replace `(first (first coll))` with `ffirst`.

```clojure
; Bad
(first (first coll))

; Good
(ffirst coll)
```

#### style/first-next
Use `fnext` instead of `(first (next coll))`.

```clojure
; Bad
(first (next coll))

; Good
(fnext coll)
```

#### style/next-first
Replace `(next (first coll))` with `nfirst`.

```clojure
; Bad
(next (first coll))

; Good
(nfirst coll)
```

#### style/next-next
Use `nnext` instead of `(next (next coll))`.

```clojure
; Bad
(next (next coll))

; Good
(nnext coll)
```

#### style/nested-addition
Flatten nested addition.

```clojure
; Bad
(+ x (+ y z))

; Good
(+ x y z)
```

#### style/nested-multiply
Flatten nested multiplication.

```clojure
; Bad
(* x (* y z))

; Good
(* x y z)
```

### Arithmetic Simplifications

#### style/plus-one
Replace `(+ x 1)` with `(inc x)`.

```clojure
; Bad
(+ x 1)
(+ 1 x)

; Good
(inc x)
```

#### style/minus-one
Replace `(- x 1)` with `(dec x)`.

```clojure
; Bad
(- x 1)

; Good
(dec x)
```

#### style/plus-zero
Simplify `(+ x 0)` to `x`.

```clojure
; Bad
(+ x 0)

; Good
x
```

#### style/minus-zero
Simplify `(- x 0)` to `x`.

```clojure
; Bad
(- x 0)

; Good
x
```

#### style/multiply-by-one
Simplify `(* x 1)` to `x`.

```clojure
; Bad
(* x 1)

; Good
x
```

#### style/multiply-by-zero
Simplify `(* x 0)` to `0`.

```clojure
; Bad
(* x 0)

; Good
0
```

### Control Flow

#### style/cond-else
Use `:else` instead of `true` in `cond`.

```clojure
; Bad
(cond
  (= x 1) :one
  true :default)

; Good
(cond
  (= x 1) :one
  :else :default)
```

#### style/prefer-condp
Convert repetitive `cond` patterns to `condp`.

```clojure
; Bad
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

**Note**: Not marked as "safe" - requires manual review.

#### style/when-do
Remove explicit `do` from `when` (implicit already).

```clojure
; Bad
(when condition
  (do
    (action-1)
    (action-2)))

; Good
(when condition
  (action-1)
  (action-2))
```

#### style/when-not-do
Remove explicit `do` from `when-not`.

```clojure
; Bad
(when-not condition
  (do
    (action)))

; Good
(when-not condition
  (action))
```

#### style/when-not-call
Use `when-not` instead of `(when (not ...))`.

```clojure
; Bad
(when (not condition)
  (action))

; Good
(when-not condition
  (action))
```

#### style/when-not-not
Simplify double negatives.

```clojure
; Bad
(when-not (not condition)
  (action))

; Good
(when condition
  (action))
```

#### style/when-not-empty?
Use `(when (seq x))` instead of `(when-not (empty? x))`.

```clojure
; Bad
(when-not (empty? coll)
  (process coll))

; Good
(when (seq coll)
  (process coll))
```

#### style/let-do
Rely on `let`'s implicit `do`.

```clojure
; Bad
(let [x 1]
  (do
    (println x)
    x))

; Good
(let [x 1]
  (println x)
  x)
```

### Data Structure Manipulation

#### style/assoc-assoc
Merge nested `assoc` calls.

```clojure
; Bad
(assoc (assoc m :a 1) :b 2)

; Good
(assoc m :a 1 :b 2)
```

#### style/update-in-assoc
Replace `(update-in coll [:a :b] assoc 5)` with `(assoc-in coll [:a :b] 5)`.

```clojure
; Bad
(update-in m [:level1 :level2] assoc :key value)

; Good
(assoc-in m [:level1 :level2 :key] value)
```

### Function Definition

#### style/def-fn
Prefer `defn` over `(def (fn ...))`.

```clojure
; Bad
(def my-func (fn [x] (inc x)))

; Good
(defn my-func [x] (inc x))
```

#### style/multiple-arity-order
Sort function arities from fewest to most arguments.

```clojure
; Bad
(defn foo
  ([a b c] ...)
  ([a] ...)
  ([a b] ...))

; Good
(defn foo
  ([a] ...)
  ([a b] ...)
  ([a b c] ...))
```

### Higher-Order Functions

#### style/mapcat-apply-apply
Replace `(apply concat (apply map x y))` with `(mapcat x y)`.

```clojure
; Bad
(apply concat (apply map f colls))

; Good
(mapcat f colls)
```

#### style/mapcat-concat-map
Replace `(apply concat (map x y))` with `(mapcat x y)`.

```clojure
; Bad
(apply concat (map f coll))

; Good
(mapcat f coll)
```

#### style/redundant-nested-call
Flatten redundant nested calls of variadic functions.

```clojure
; Bad
(str (str a b) c)

; Good
(str a b c)
```

### Interop and Java Integration

#### style/tostring
Use `str` instead of `.toString`.

```clojure
; Bad
(.toString obj)

; Good
(str obj)
```

#### style/new-object
Prefer dot notation `ClassName.` over `new` special form.

```clojure
; Bad
(new Object)

; Good
(Object.)
```

#### style/prefer-clj-math
Use `clojure.math` instead of `Math/` interop (Clojure 1.11+).

```clojure
; Bad
(Math/sqrt 16)

; Good
(require '[clojure.math :as math])
(math/sqrt 16)
```

#### style/prefer-clj-string
Use `clojure.string` functions instead of string method interop.

```clojure
; Bad
(.toLowerCase s)
(.toUpperCase s)

; Good
(require '[clojure.string :as str])
(str/lower-case s)
(str/upper-case s)
```

### Other Patterns

#### style/not-nil?
Use `some?` instead of `(not (nil? x))`.

```clojure
; Bad
(not (nil? x))

; Good
(some? x)
```

#### style/not-some-pred
Use `not-any?` instead of `(not (some pred coll))`.

```clojure
; Bad
(not (some even? coll))

; Good
(not-any? even? coll)
```

#### style/prefer-boolean
Use `boolean` instead of `(if x true false)`.

```clojure
; Bad
(if x true false)

; Good
(boolean x)
```

#### style/prefer-for-with-literals
Use `for` with literals instead of `map` with anonymous functions.

```clojure
; Bad
(map #(hash-map :key %) coll)

; Good
(for [x coll] {:key x})
```

#### style/prefer-vary-meta
Use `vary-meta` instead of manually updating metadata.

```clojure
; Bad
(with-meta x (assoc (meta x) :key val))

; Good
(vary-meta x assoc :key val)
```

#### style/redundant-let
Merge nested `let` blocks.

```clojure
; Bad
(let [x 1]
  (let [y 2]
    (+ x y)))

; Good
(let [x 1
      y 2]
  (+ x y))
```

#### style/redundant-regex-constructor
Remove `re-pattern` wrapping regex literals.

```clojure
; Bad
(re-pattern #"pattern")

; Good
#"pattern"
```

#### style/set-literal-as-fn
Use `case` instead of sets as functions.

```clojure
; Bad
(#{:a :b :c} x)

; Good
(case x
  (:a :b :c) true
  false)
```

#### style/trivial-for
Replace simple `for` loops with `map`.

```clojure
; Bad
(for [x coll] (inc x))

; Good
(map inc coll)
```

#### style/useless-do
Remove unnecessary `do` wrappers.

```clojure
; Bad
(do x)

; Good
x
```

#### style/is-eq-order
Place expected value first in test assertions.

```clojure
; Bad
(is (= actual expected))

; Good
(is (= expected actual))
```

#### style/prefixed-libspecs
Avoid prefixed require specs for clarity.

```clojure
; Bad
(:require [clojure [string :as str] [set :as set]])

; Good
(:require [clojure.string :as str]
          [clojure.set :as set])
```

#### style/prefix-libspecs
Expand collapsed namespace prefixes in requires.

```clojure
; Bad
(:require [my.app [core :as core] [util :as util]])

; Good
(:require [my.app.core :as core]
          [my.app.util :as util])
```

## Naming Rules

Naming rules enforce identifier naming conventions.

### naming/conventional-aliases

**Status**: Enabled by default | Safe | No autocorrect

Use community-standard aliases for core libraries.

```clojure
; Bad
(:require [clojure.string :as string])

; Good
(:require [clojure.string :as str])
```

**Common aliases**: `async`, `mat`, `csv`, `xml`, `edn`, `io`, `pp`, `set`, `str`, `walk`, `zip`

### naming/conversion-functions

**Status**: Enabled by default | Not safe | No autocorrect

Use arrow notation (`->`) instead of `to` in conversion function names.

```clojure
; Bad
(defn f-to-c ...)

; Good
(defn f->c ...)
```

**Note**: Only warns when `-to-` lacks surrounding hyphens.

### naming/lisp-case

**Status**: Enabled by default | Not safe | No autocorrect

Enforce lisp-case (hyphen-separated lowercase) for functions and variables.

```clojure
; Bad
(def someVar ...)
(def some_fun ...)

; Good
(def some-var ...)
(defn some-fun ...)
```

**Exceptions**: Skips names containing `->` or ending in `?`.

### naming/predicate

**Status**: Enabled by default | Not safe | No autocorrect

Boolean-returning functions should end with `?`.

```clojure
; Bad
(defn palindrome-p ...)
(defn is-palindrome ...)

; Good
(defn palindrome? ...)
```

### naming/record-name

**Status**: Enabled by default | Safe | No autocorrect

Records must use PascalCase naming convention.

```clojure
; Bad
(defrecord foo [a b c])
(defrecord foo-bar [a b c])

; Good
(defrecord Foo [a b c])
(defrecord FooBar [a b c])
```

### naming/single-segment-namespace

**Status**: Enabled by default | Safe | No autocorrect

Prevent naming conflicts by requiring multi-segment namespaces.

```clojure
; Bad
(ns simple)

; Good
(ns noahtheduke.simple)
```

**Rationale**: "Namespaces exist to disambiguate names. Using a single segment namespace puts you in direct conflict with everyone else."

## Performance Rules

**Note**: Performance rules are **disabled by default** as they are more contentious. Enable selectively based on your team's needs.

### performance/assoc-many

**Status**: Disabled by default | Safe | Autocorrectable

Multiple key-value pairs in `assoc` are slower than chained invocations.

```clojure
; Current
(assoc m :k1 1 :k2 2)

; Suggested
(-> m
    (assoc :k1 1)
    (assoc :k2 2))
```

### performance/avoid-satisfies

**Status**: Disabled by default | Safe | No autocorrect

Avoid use of `satisfies?` as it is extremely slow. Restructure code to use protocols or other polymorphic alternatives.

### performance/dot-equals

**Status**: Disabled by default | Safe | Autocorrectable

Direct Java method calls outperform generic equality for string literals.

```clojure
; Current
(= "foo" s)

; Suggested
(.equals "foo" s)
; or
(String/equals "foo" s)
```

### performance/get-in-literals

**Status**: Disabled by default | Safe | Autocorrectable

Keyword-only path access is faster without `get-in` overhead.

```clojure
; Current
(get-in m [:key1 :key2])

; Suggested
(-> m :key1 :key2)
```

### performance/get-keyword

**Status**: Disabled by default | Safe | Autocorrectable

Keywords as functions outperform polymorphic `get` for map access.

```clojure
; Current
(get m :key)

; Suggested
(:key m)
```

### performance/into-transducer

**Status**: Disabled by default | Safe | Autocorrectable

The 4-arity `into` form efficiently applies transducers between operations.

```clojure
; Current
(into [] (map inc (range 100)))

; Suggested
(into [] (map inc) (range 100))
```

**Configuration**: Supports custom transducers via `:fn-0-arg` and `:fn-1-arg` sets.

### performance/single-literal-merge

**Status**: Disabled by default | Safe | Autocorrectable

Merge is inherently slow when combining single map literals.

```clojure
; Current
(merge {:a 1} {:b 2})

; Suggested
{:a 1 :b 2}
; or for multiple keys
(-> {}
    (assoc :a 1)
    (assoc :b 2))
```

**Configuration**: Supports dynamic, single, or multiple style options.

## Metrics Rules

**Note**: Metrics rules are **disabled by default**. Enable them to enforce code quality thresholds.

### metrics/fn-length

**Status**: Disabled by default | Safe | No autocorrect

Enforce a maximum line count for `defn`-defined functions to improve readability.

**Default limit**: 10 lines

**Configuration options**:
- `:chosen-style` (default: `:body`) - Choose between:
  - `:body`: Measure from parameters onward
  - `:defn`: Measure entire form including function name
- `:length` (default: `10`) - Set maximum line count

```clojure
; Example configuration
{:metrics/fn-length {:enabled true
                     :chosen-style :body
                     :length 15}}
```

**Rationale**: "Longer functions are harder to read and should be split into smaller-purpose functions that are composed."

### metrics/parameter-count

**Status**: Disabled by default | Safe | No autocorrect

Discourage function signatures with excessive positional parameters.

**Default limit**: 4 parameters

**Configuration options**:
- `:chosen-style` (default: `:positional`) - Choose between:
  - `:positional`: Excludes `& args` from count
  - `:include-rest`: Includes rest parameters in count
- `:count` (default: `4`) - Set maximum positional parameter count

```clojure
; Example configuration
{:metrics/parameter-count {:enabled true
                           :chosen-style :positional
                           :count 3}}
```

**Note**: Functions with multiple arities will have each arity checked.

## Rule Configuration Examples

### Enable/Disable Individual Rules

```clojure
{:rules {:lint/eq-nil {:enabled false}
         :style/prefer-condp {:enabled true}}}
```

### Disable Entire Categories

```clojure
{:rules {:performance {:enabled false}
         :metrics {:enabled false}}}
```

### Configure Rule Options

```clojure
{:rules {:metrics/fn-length {:enabled true
                             :chosen-style :body
                             :length 15}
         :metrics/parameter-count {:enabled true
                                   :chosen-style :include-rest
                                   :count 3}
         :performance/into-transducer {:enabled true
                                       :fn-0-arg #{list vector set}
                                       :fn-1-arg #{map filter remove}}}}
```

### Inline Rule Disabling

Disable rules for specific forms in code:

```clojure
; Disable single rule for one form
#_:splint/disable (+ 1 x)

; Disable multiple rules for one form
#_{:splint/disable [style/plus-one lint/redundant-call]}
(do (+ 1 x))
```

## Auto-Generate Configuration

Use the `--auto-gen-config` flag to generate a `.splint.edn` file that disables all currently failing rules. Each entry includes:
- Diagnostics count
- Rule description
- Available configuration styles (for reference)

```bash
splint --auto-gen-config
```

This creates a baseline configuration that you can then selectively enable rules from as you fix issues.
