# Stuart Sierra's Clojure Do's and Don'ts

Comprehensive reference of rules from Stuart Sierra's "Clojure Don'ts" blog series. These principles focus on avoiding common pitfalls and writing idiomatic, maintainable Clojure code.

**Source:** https://stuartsierra.com/tag/dos-and-donts

---

## 1. Don't Mix Lazy Effects

**Priority:** Critical - "probably my number one Clojure Don't"

### The Rule

Never mix side effects with lazy sequence operations. Laziness allows you to express computations without specifying *when* they should happen, which is fundamentally incompatible with side effects that must occur at specific times.

### The Problem

Lazy sequence functions like `map` don't guarantee execution timing:

```clojure
;; BAD - unpredictable execution
(take 5 (map prn (range 10)))
;; Prints all 10 numbers due to "chunked sequence" optimization (batches of 32)
;; not just 5 as you might expect

;; BAD - may never execute
(do (map prn [0 1 2 3 4 5 6 7 8 9 10])
    (println "Hello, world!"))
;; Only prints "Hello, world!" because the map is never forced
```

### The Solution

For side-effecting operations, use eager evaluation:

```clojure
;; GOOD - use doseq (best default for side effects)
(doseq [item items]
  (save-to-db! item))

;; GOOD - use run! (Clojure 1.7+)
(run! println items)

;; GOOD - use reduce or transduce
(reduce (fn [_ item] (prn item)) nil items)

;; GOOD - transducers with predictable evaluation
(transduce (comp (take 5) (map prn)) conj [] (range 10))
;; Prints exactly 5 elements
```

### When to Use Each

- **`doseq`**: Best default; clearly signals side effects with imperative syntax
- **`run!`**: Cleaner than `(dorun (map ...))` for simple cases
- **`reduce`/`transduce`**: When combining side effects with accumulation
- **`doall`/`dorun`**: Only when forced to work with existing lazy code

### Why This Matters

Side effects that don't execute, execute late, or execute in batches cause:
- Database operations that silently fail
- Logging that doesn't happen when expected
- Resource cleanup that never occurs
- Unpredictable timing bugs that are hard to reproduce

---

## 2. Don't Create Redundant Map Functions

### The Rule

Don't create a separate function to apply a single-item operation across a collection. If you have an operation on a single object, you don't need to define another version just to operate on a collection.

### The Anti-Pattern

```clojure
;; BAD - redundant wrapper
(defn process-thing [thing]
  ;; process one thing
  ...)

(defn process-many-things [things]
  (map process-thing things))
```

### Why It's Wrong

The idiom "map a function over a collection" is universal in Clojure. Any Clojure programmer should be able to write it without thinking twice. The wrapper adds:
- Unnecessary API surface
- Extra names to remember
- Maintenance burden with no benefit
- False impression of doing something special

### The Solution

```clojure
;; GOOD - single-item function only
(defn process-thing [thing]
  ;; process one thing
  ...)

;; Callers use map directly
(map process-thing things)
```

### The Exception

Only create a batch version if there's something genuinely special about processing groups:

```clojure
;; GOOD - batch version has real optimization
(defn process-thing [thing]
  (db/insert! thing))

(defn process-many-things [things]
  ;; Genuinely more efficient batch insert
  (db/batch-insert! things))
```

### Going Further

Even if you initially only have a collection-processing function, refactoring it into a single-item function increases flexibility:

```clojure
;; Starting point
(defn process-things [things]
  (doseq [thing things]
    (do-something thing)))

;; Better - refactored to single-item
(defn process-thing [thing]
  (do-something thing))

;; Now callers can:
(map process-thing things)           ; map
(run! process-thing things)           ; side effects
(filter valid? (map process-thing things))  ; compose
```

---

## 3. Don't Use Single-Branch If

### The Rule

An `if` should always have both 'then' and 'else' branches. Use `when` for a condition which should return `nil` in the negative case.

### The Problem

Three approaches exist for single-branch conditionals:

```clojure
;; Option 1: when
(when (condition? x)
  (then-expression x))

;; Option 2: if with explicit nil
(if (condition? x)
  (then-expression x)
  nil)

;; Option 3: if without else
(if (condition? x)
  (then-expression x))
```

### Why Use `when`

Based on readability and expectations:

1. **Pattern Recognition**: Most Clojure code uses `if` with complete then/else branches. Readers expect this pattern.

2. **Visual Confusion**: Encountering an `if` without an else branch causes momentary confusion—the eye searches for missing code.

3. **Looks Like an Error**: Explicitly returning `nil` looks like a mistake, since many Clojure expressions naturally default to `nil`.

4. **Inconsistent Convention**: The pattern becomes inconsistent with Clojure community standards.

### The Solution

```clojure
;; GOOD - clear single-branch intent
(when (valid? data)
  (process data)
  (save data))

;; GOOD - if with both branches
(if (valid? data)
  (process-and-save data)
  (handle-invalid data))

;; BAD - single-branch if
(if (valid? data)
  (process-and-save data))

;; BAD - explicit nil return
(if (valid? data)
  (process-and-save data)
  nil)
```

### Addressing the Side Effects Argument

Some argue `when` is only for side effects. Sierra prioritizes **consistent, scannable code** over ideological purity. Using `when` for single-branch conditionals creates uniform, predictable patterns that improve comprehension.

---

## 4. Don't Use Polymorphism Where It Doesn't Exist

### The Rule

Don't use polymorphism where it doesn't exist. The fundamental principle is that polymorphism should only be employed when callers genuinely don't know which implementation will execute—when substitutability actually matters.

### The Problem

A common mistake is using protocols or multimethods even when each call site unambiguously knows which method will run. This creates false abstraction, hiding tight coupling behind the appearance of decoupling.

### The Anti-Pattern

```clojure
;; BAD - false polymorphism
(defprotocol Processor
  (process [this]))

(defrecord FooProcessor []
  Processor
  (process [this] ...))

(defrecord BarProcessor []
  Processor
  (process [this] ...))

;; Call site always knows the type
(defn handle-foo [x]
  (process x)  ; x is always FooProcessor
  ;; Code relies on FooProcessor-specific behavior
  (:foo-specific-field x))
```

This breaks the abstraction contract—you're coupled to specific implementations while pretending you're not.

### Why It's Wrong

1. **False Decoupling**: The protocol suggests flexibility that doesn't exist
2. **Hidden Dependencies**: Real coupling is obscured by abstraction
3. **Maintenance Burden**: Extra complexity with no benefit
4. **Broken Liskov Substitution**: Implementations aren't truly substitutable

### The Solution

Use distinct named functions that make coupling explicit:

```clojure
;; GOOD - honest, explicit naming
(defn process-foo [foo]
  ;; Process foo-specific data
  ...)

(defn process-bar [bar]
  ;; Process bar-specific data
  ...)

(defn handle-foo [x]
  (process-foo x)  ; Coupling is obvious and honest
  ...)
```

### When Polymorphism Is Appropriate

Use protocols/multimethods when:

```clojure
;; GOOD - genuine polymorphism
(defprotocol Saveable
  (save [this]))

;; Caller doesn't know or care about type
(defn persist-entities [entities]
  (doseq [entity entities]
    (save entity)))  ; Works with any Saveable

;; True substitutability
(extend-protocol Saveable
  User
  (save [user] (save-to-users-table user))

  Order
  (save [order] (save-to-orders-table order)))
```

### The Liskov Substitution Principle

Sierra invokes this principle: "If you cannot substitute one implementation of a protocol for another, it's not a good abstraction." Polymorphism should enable genuine flexibility; when it doesn't, simpler mechanisms are more appropriate.

---

## 5. Don't Use `as->` Outside Threading Context

### The Rule

Never use `as->` by itself outside of a `->` threading context. The macro's argument order only makes sense within `->`.

### Why This Matters

`as->` places the threaded value **second** in its parameters, signaling it's "meant to be used in combination with `->`." Using it standalone violates the expected pattern and reduces readability.

### Good Use Case

When you have functions following a consistent pattern but need to call one library function with non-standard argument placement:

```clojure
;; GOOD - as-> within threading chain
(-> context
    (one ...)
    (two ...)
    (as-> ctx (irritating-library-fn arg1 ctx arg3))
    (three ...))
```

The `as->` temporarily breaks the chain to accommodate the awkward function, then resumes the pattern.

### Bad Use Case

```clojure
;; BAD - standalone as->
(as-> (initial-value) it
      (one it ...)
      (two it ...))
```

Sierra notes this is "only marginally better than `-$>`" and lacks clarity about the symbol's meaning.

### Better Alternatives

```clojure
;; GOOD - use let for clarity
(let [value (initial-value)
      value (one value ...)
      value (two value ...)]
  value)

;; GOOD - refactor to use standard threading
(-> (initial-value)
    (one ...)
    (two ...))

;; GOOD - use cond-> for conditional threading
(cond-> (initial-value)
  (some-test?) (one ...)
  (other-test?) (two ...))
```

### When to Consider Refactoring

If you need `as->` frequently, consider:
1. Wrapping the awkward function with standard argument order
2. Using a different API if available
3. Restructuring the data flow

---

## 6. Don't Use `#()` for Multi-Expression Functions

### The Rule

The reader macro `#()` creates an anonymous function whose body is a **single expression**. Multi-expression bodies require `fn`.

### The Problem

```clojure
;; BAD - #() requires single expression
#(do
   (println %)
   (* % 2))

;; BAD - implicit do doesn't work
#((println %)
  (* % 2))
```

### The Solution

```clojure
;; GOOD - single expression
#(* % 2)

;; GOOD - fn for multiple expressions
(fn [x]
  (println x)
  (* x 2))

;; GOOD - extract to named function
(defn double-with-logging [x]
  (println x)
  (* x 2))

(map double-with-logging numbers)
```

### When to Use Each

- **`#()`**: Single, simple expressions (especially inline)
- **`fn`**: Multiple expressions, multiple arities, destructuring
- **`defn`**: Reusable, named functions (most cases)

---

## 7. Don't Use Numbered Parameters Carelessly

### The Rule

While `#()` supports numbered parameters (`%1`, `%2`, `%3`), use them carefully. For functions with multiple parameters or complex logic, `fn` with named parameters improves readability.

### The Problem

```clojure
;; BAD - unclear what parameters mean
#(process %1 %2 %3)

;; BAD - complex logic with %
#(if (valid? %1)
   (combine %1 %2)
   (default %2 %3))
```

### The Solution

```clojure
;; GOOD - simple single-parameter use
#(* % 2)
#(.toLowerCase %)

;; GOOD - named parameters for clarity
(fn [user order status]
  (if (valid? user)
    (combine user order)
    (default order status)))
```

### Guidelines

Use `#()` when:
- Single parameter (`%`)
- Very short, obvious expression
- Would be clear to any reader

Use `fn` when:
- Multiple parameters need names
- Logic is non-trivial
- Reading requires mental effort

---

## Summary: Quick Reference

**Critical Rules:**
1. Never mix lazy sequences with side effects (use `doseq`, `run!`, `reduce`)
2. Don't create redundant batch functions (let callers use `map`)
3. Use `when` for single-branch conditionals (not `if`)
4. Only use polymorphism when substitutability matters
5. Never use `as->` outside `->` threading chains
6. Use `#()` only for single-expression functions
7. Use named parameters when clarity demands it

**Core Philosophy:**
- Make coupling explicit, not hidden
- Use the most specific construct for your intent
- Prioritize readability and convention
- Avoid false abstractions
- Choose clarity over cleverness

**When in Doubt:**
- More explicit is better than more clever
- Standard patterns over custom solutions
- Named functions over anonymous when reused
- Honest coupling over false decoupling
