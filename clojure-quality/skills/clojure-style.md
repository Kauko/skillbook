# Clojure Style Guide

Use when writing or reviewing Clojure code. This skill enforces coding conventions from Stuart Sierra's Do's and Don'ts and the Clojure Style Guide, plus custom Metosin conventions.

Apply these rules automatically when writing Clojure code. When reviewing code, cite specific rules and explain why they matter.

## Naming Conventions

### Functions and Variables: lisp-case (kebab-case)

```clojure
;; GOOD
(defn calculate-total-price [items]
  ...)

(let [user-name "Alice"
      order-count 5]
  ...)

;; BAD
(defn calculateTotalPrice [items]  ; camelCase
  ...)

(defn calculate_total_price [items]  ; snake_case
  ...)
```

**Why:** Kebab-case is the universal Clojure community standard. It's readable and distinguishes Clojure from Java interop.

### Predicates: End with `?`

```clojure
;; GOOD
(defn valid-email? [s]
  (re-matches #".+@.+\..+" s))

(when (empty? coll)
  ...)

;; BAD
(defn valid-email [s]  ; Missing ?
  ...)

(defn is-empty [coll]  ; Don't use "is-" prefix
  ...)
```

**Why:** The `?` suffix immediately signals a boolean return value. Avoid verbose `is-` prefixes.

### Unsafe Operations: End with `!`

```clojure
;; GOOD
(swap! atom-state update :counter inc)
(reset! atom-state {})
(send! agent-ref process)

;; These have ! because they mutate or do I/O
(defn save-to-db! [data]
  (jdbc/insert! db :users data))
```

**Why:** The `!` signals side effects or mutations, warning readers to be careful in STM contexts.

### Dynamic Vars: Wrap with `*earmuffs*`

```clojure
;; GOOD
(def ^:dynamic *db-connection* nil)
(def ^:dynamic *current-user* nil)

(binding [*db-connection* conn]
  ...)

;; BAD
(def ^:dynamic db-connection nil)  ; Missing earmuffs
```

**Why:** Earmuffs make dynamic vars immediately recognizable and prevent confusion with regular vars.

### Types, Records, Protocols: CapitalCase

```clojure
;; GOOD
(defprotocol DataStore
  (save [this data])
  (load [this id]))

(defrecord User [id name email])

(deftype CustomQueue [items])

;; BAD
(defprotocol data-store  ; Should be CapitalCase
  ...)

(defrecord user [id name email])  ; Should be User
```

**Why:** CapitalCase distinguishes types from functions and aligns with Java conventions for interop.

### Unused Bindings: Use `_`

```clojure
;; GOOD
(let [[first-item _ third-item] collection]
  (process first-item third-item))

(fn [_ event]  ; Ignore first argument
  (handle-event event))

;; ACCEPTABLE (when you want the name for clarity)
(let [[first-item _second-item third-item] collection]
  ...)
```

**Why:** `_` signals intentional ignoring. Prefix with `_` when you want a descriptive name but won't use the value.

## Code Organization

### One Namespace Per File

```clojure
;; GOOD: src/myapp/orders/core.clj
(ns myapp.orders.core
  ...)

;; BAD: Don't put multiple namespaces in one file
(ns myapp.orders.core ...)
(ns myapp.orders.utils ...)  ; Should be separate file
```

**Why:** One-to-one mapping between files and namespaces makes code easy to locate.

### Multi-Segment Namespace Names

```clojure
;; GOOD
(ns myapp.orders.core)
(ns mycompany.myapp.auth)

;; BAD
(ns orders)  ; Single segment
(ns core)    ; Too generic
```

**Why:** Multi-segment names avoid collisions and provide context. Single-segment names are reserved for libraries.

### Namespace Declaration Order

```clojure
;; GOOD
(ns myapp.core
  "Namespace docstring"
  (:require [clojure.string :as str]
            [clojure.set :as set]
            [myapp.config :as config]
            [myapp.db :as db])
  (:import [java.time Instant]
           [java.util UUID]))

;; BAD - imports before requires
(ns myapp.core
  (:import [java.util UUID])
  (:require [clojure.string :as str]))
```

**Why:** Standard order is: docstring, `:require`, `:import`. Consistent ordering improves readability.

### Top-Level Form Spacing

```clojure
;; GOOD - single blank line between top-level forms
(defn foo []
  :foo)

(defn bar []
  :bar)

(defn baz []
  :baz)

;; BAD - no spacing
(defn foo []
  :foo)
(defn bar []
  :bar)

;; BAD - too much spacing
(defn foo []
  :foo)


(defn bar []
  :bar)
```

**Why:** Single blank lines between definitions improve readability without excessive whitespace.

### No Internal Blank Lines in Definitions

```clojure
;; GOOD
(defn process-order
  [order]
  (let [total (calculate-total order)]
    (if (valid? total)
      (save! order)
      (throw (ex-info "Invalid total" {:total total})))))

;; BAD - unnecessary blank lines inside function
(defn process-order
  [order]

  (let [total (calculate-total order)]

    (if (valid? total)
      (save! order)
      (throw (ex-info "Invalid total" {:total total})))))
```

**Why:** Blank lines should separate top-level forms, not clutter function internals.

## Idiomatic Patterns

### Threading Macros: Reduce Nesting

```clojure
;; GOOD - use threading to show data flow
(-> data
    (assoc :id (UUID/randomUUID))
    (update :timestamp #(or % (Instant/now)))
    (dissoc :internal))

(->> users
     (filter active?)
     (map :email)
     (take 10))

;; BAD - deeply nested
(dissoc
  (update
    (assoc data :id (UUID/randomUUID))
    :timestamp #(or % (Instant/now)))
  :internal)
```

**Why:** Threading macros make data transformations readable by showing the flow linearly.

### Use `when` for Single-Branch Conditionals

```clojure
;; GOOD
(when (logged-in? user)
  (log-access user)
  (fetch-data user))

;; BAD - unnecessary if
(if (logged-in? user)
  (do
    (log-access user)
    (fetch-data user))
  nil)
```

**Why:** `when` has implicit `do` and nil-else, making single-branch conditionals cleaner.

**Related: Don't use single-branch `if`** (Stuart Sierra)

### Use `seq` for Empty Collection Tests

```clojure
;; GOOD
(when (seq collection)
  (process collection))

(if (seq items)
  (first items)
  default-value)

;; BAD
(when (not (empty? collection))
  (process collection))

(if (> (count items) 0)
  (first items)
  default-value)
```

**Why:** `seq` returns nil for empty collections, enabling idiomatic nil-punning. It's also more efficient than `count`.

### ALWAYS Use Fallback with `get` (Metosin Convention)

```clojure
;; GOOD - explicit fallback makes nil-handling intentional
(get my-map :id ::not-found)
(get my-map :name :anonymous)
(get my-map :count 0)

;; BAD - ambiguous whether nil is a valid value or missing key
(get my-map :id)
```

**Why:** Explicit fallback makes nil-handling intentional and aids debugging. Without fallback, you can't distinguish between "key missing" and "key present with nil value". The fallback documents expected behavior.

**Exceptions:**
- When nil is a valid, expected value
- In pure predicates where nil test is explicit: `(some? (get m :k))`

### Combine Binding with Conditionals

```clojure
;; GOOD
(if-let [user (find-user id)]
  (greet user)
  (create-guest))

(when-let [config (load-config path)]
  (validate config)
  (apply-config config))

;; BAD - unnecessary two-step
(let [user (find-user id)]
  (if user
    (greet user)
    (create-guest)))
```

**Why:** `if-let` and `when-let` combine binding with nil-checking, reducing boilerplate.

### Prefer Higher-Order Functions over `loop/recur`

```clojure
;; GOOD
(map #(* % 2) numbers)
(filter even? numbers)
(reduce + 0 numbers)

;; BAD - unnecessary loop
(loop [nums numbers
       result []]
  (if (seq nums)
    (recur (rest nums) (conj result (* 2 (first nums))))
    result))
```

**Why:** Higher-order functions are more declarative and often optimized. Use `loop/recur` only when necessary for performance or complex iteration.

### Collections as Functions

```clojure
;; GOOD - keywords and maps as functions
(:name user)
(user :name)
(#{:admin :moderator} role)

;; ACCEPTABLE but less idiomatic
(get user :name)
(contains? #{:admin :moderator} role)
```

**Why:** Keywords and collections as functions is idiomatic Clojure. However, see "ALWAYS use fallback with get" rule above.

### Prefer `condp` for Repeated Predicates (Stuart Sierra)

```clojure
;; GOOD
(condp = status
  :pending (handle-pending order)
  :approved (handle-approved order)
  :rejected (handle-rejected order)
  (handle-unknown order))

;; BAD - repetitive
(cond
  (= status :pending) (handle-pending order)
  (= status :approved) (handle-approved order)
  (= status :rejected) (handle-rejected order)
  :else (handle-unknown order))
```

**Why:** `condp` eliminates repetition when testing the same value multiple times.

### Avoid Lazy Side Effects (Stuart Sierra)

```clojure
;; BAD - side effects in lazy sequence
(map #(println %) items)  ; Might not execute!
(for [item items]
  (save-to-db! item))     ; Dangerous!

;; GOOD - force realization with run!, doseq, doall
(run! println items)
(doseq [item items]
  (save-to-db! item))
(doall (map process-and-save items))
```

**Why:** Lazy sequences delay evaluation. Side effects in lazy sequences may never execute or execute at unexpected times. Use `run!`, `doseq`, or `doall` to force realization.

### Single-Expression Functions: Use `#()` Reader Macro

```clojure
;; GOOD - concise for simple functions
(map #(* % 2) numbers)
(filter #(> % 10) numbers)
(sort-by #(.toLowerCase %) strings)

;; BAD - overly verbose
(map (fn [x] (* x 2)) numbers)

;; BAD - #() must be single expression (Stuart Sierra)
#(do
   (println %)
   (* % 2))  ; Should use (fn [x] ...)
```

**Why:** `#()` is concise for simple, single-expression functions. For multi-expression bodies, use `fn`.

### Use Polymorphism for Consistent Interfaces (Stuart Sierra)

```clojure
;; GOOD - polymorphic protocol
(defprotocol Saveable
  (save [this]))

(extend-protocol Saveable
  User
  (save [user] (save-user-to-db user))

  Order
  (save [order] (save-order-to-db order)))

;; Now callers don't need to know the type
(save entity)

;; BAD - type-specific functions force caller to know types
(defn save-user [user] ...)
(defn save-order [order] ...)

;; Caller must know type
(if (user? entity)
  (save-user entity)
  (save-order entity))
```

**Why:** Polymorphism decouples callers from implementation details, making code more flexible and maintainable.

## Documentation

### Docstrings for Public Functions

```clojure
;; GOOD
(defn calculate-discount
  "Calculates discount amount for an order.

  Args:
    order - Map containing :total and :customer-type
    coupon - Optional coupon code string

  Returns:
    Discount amount as decimal (0.0 to 1.0)

  Example:
    (calculate-discount {:total 100 :customer-type :premium} \"SAVE20\")"
  [order coupon]
  ...)

;; BAD - missing or inadequate docstring
(defn calculate-discount [order coupon]
  ...)
```

**Why:** Docstrings provide inline documentation. Include parameters, return values, and examples for complex functions.

### Comment Hierarchy

```clojure
;;;; Section: Order Processing
;;;; This section handles the complete order lifecycle

;;; Validation
;;; Functions for validating orders before processing

(defn valid-order?
  "Checks if order has required fields"
  [order]
  ;; TODO: Add inventory check
  (and (contains? order :items)
       (contains? order :customer)))  ; Inline comment
```

**Comment levels:**
- `;;;;` - Major section headings
- `;;;` - Subsection descriptions
- `;;` - Function-level comments (above function)
- `;` - Inline comments (end of line)

**Why:** Consistent comment structure makes code organization clear.

### Prioritize Self-Explanatory Code

```clojure
;; GOOD - names explain intent
(defn active-premium-users [users]
  (->> users
       (filter :active?)
       (filter #(= :premium (:tier %)))))

;; BAD - comments compensate for poor names
(defn get-users [users]
  ;; Filter for active users
  ;; Then filter for premium tier
  (->> users
       (filter :active?)
       (filter #(= :premium (:tier %)))))
```

**Why:** Clear names eliminate the need for most comments. Use comments for "why", not "what".

## Code Formatting

### Line Length: 80-120 Characters

```clojure
;; GOOD - within limit
(defn process-payment [order payment-method customer]
  ...)

;; ACCEPTABLE - slightly over for readability
(defn calculate-shipping-cost [order destination-address shipping-method warehouse-location]
  ...)

;; BAD - too long, should be split
(defn calculate-total-with-discounts-and-taxes [order customer-loyalty-level coupon-codes regional-tax-rates]
  ...)
```

**Why:** 80 characters is ideal; 120 is acceptable. Longer lines hurt readability.

### Indentation: 2 Spaces, Never Tabs

```clojure
;; GOOD
(defn my-function
  [arg]
  (let [x 1
        y 2]
    (+ x y arg)))

;; BAD - inconsistent indentation
(defn my-function
[arg]
    (let [x 1
      y 2]
  (+ x y arg)))
```

**Why:** Consistent 2-space indentation is universal in Clojure. Use tools like cljfmt to enforce.

### Semantic Indentation

```clojure
;; GOOD - forms align semantically
(when (valid? data)
  (process data)
  (save data))

(let [users (fetch-users)
      orders (fetch-orders users)]
  (generate-report users orders))

;; BAD - inconsistent alignment
(when (valid? data)
(process data)
(save data))
```

**Why:** Semantic indentation shows structure. Forms at the same level align vertically.

## Error Handling

### Use `ex-info` for Rich Exceptions

```clojure
;; GOOD - structured error data
(when-not (valid? order)
  (throw (ex-info "Invalid order"
                  {:order order
                   :errors (validate order)
                   :type ::validation-error})))

;; BAD - string-only exception
(when-not (valid? order)
  (throw (Exception. "Invalid order")))
```

**Why:** `ex-info` attaches structured data to exceptions, enabling better error handling and debugging.

### Descriptive Error Messages

```clojure
;; GOOD
(when-not (contains? order :customer-id)
  (throw (ex-info "Order missing required field :customer-id"
                  {:order order})))

;; BAD
(when-not (contains? order :customer-id)
  (throw (ex-info "Invalid order" {:order order})))
```

**Why:** Specific error messages speed up debugging. Say what's wrong, not just that something is wrong.

## Performance Considerations

### Use Transients for Large Batch Updates

```clojure
;; GOOD - transient for many updates
(defn build-large-map [items]
  (persistent!
   (reduce (fn [m item]
             (assoc! m (:id item) item))
           (transient {})
           items)))

;; BAD - persistent updates in tight loop (slow for large n)
(defn build-large-map [items]
  (reduce (fn [m item]
            (assoc m (:id item) item))
          {}
          items))
```

**Why:** Transients provide near-mutable performance while maintaining immutability guarantees.

### Prefer Direct Java Calls for Tight Loops

```clojure
;; GOOD - direct Java for performance-critical code
(defn process-numbers [numbers]
  (loop [i 0
         result 0.0]
    (if (< i (count numbers))
      (recur (unchecked-inc i)
             (unchecked-add result (.doubleValue (nth numbers i))))
      result)))

;; NORMALLY GOOD - idiomatic but slower
(reduce + 0.0 numbers)
```

**Why:** For performance-critical code, direct Java calls and type hints eliminate reflection. Profile first.

## Testing

### Descriptive Test Names

```clojure
;; GOOD
(deftest calculate-discount-applies-coupon-percentage
  (is (= 20.0 (calculate-discount {:total 100} "SAVE20"))))

(deftest calculate-discount-returns-zero-for-invalid-coupon
  (is (= 0.0 (calculate-discount {:total 100} "INVALID"))))

;; BAD
(deftest test1
  (is (= 20.0 (calculate-discount {:total 100} "SAVE20"))))
```

**Why:** Descriptive test names document expected behavior and make failures easier to diagnose.

### Use `testing` for Context

```clojure
;; GOOD
(deftest order-validation
  (testing "valid orders"
    (is (valid-order? {:customer-id 1 :items [{:id 1}]})))

  (testing "missing customer-id"
    (is (not (valid-order? {:items [{:id 1}]}))))

  (testing "missing items"
    (is (not (valid-order? {:customer-id 1})))))
```

**Why:** `testing` blocks provide context for assertions, making test output more informative.

## Quick Reference

**Do:**
- ✓ Use kebab-case for functions/variables
- ✓ End predicates with `?`
- ✓ End unsafe operations with `!`
- ✓ Use `*earmuffs*` for dynamic vars
- ✓ Use `when` for single-branch conditionals
- ✓ Use `seq` to test for empty collections
- ✓ **Always use fallback with `get`**
- ✓ Prefer threading macros for data flow
- ✓ Use higher-order functions over `loop/recur`
- ✓ Force realization of lazy sequences with side effects
- ✓ Write docstrings for public functions
- ✓ Use polymorphism for consistent interfaces

**Don't:**
- ✗ Use camelCase or snake_case
- ✗ Use `is-` prefix for predicates
- ✗ Use single-segment namespaces
- ✗ Use single-branch `if` (use `when`)
- ✗ Use `(not (empty? coll))` (use `seq`)
- ✗ Use `get` without fallback
- ✗ Combine laziness with side effects
- ✗ Use `#()` for multi-expression functions
- ✗ Write type-specific functions (use protocols)

## Additional Resources

**Stuart Sierra's Do's and Don'ts:**
- https://stuartsierra.com/tag/dos-and-donts
- Lazy Effects: https://stuartsierra.com/2015/08/25/clojure-donts-lazy-effects
- Redundant Map: https://stuartsierra.com/2015/04/26/clojure-donts-redundant-map
- Single-Branch If: https://stuartsierra.com/2015/06/03/clojure-donts-single-branch-if
- Non-Polymorphism: https://stuartsierra.com/2015/05/17/clojure-donts-non-polymorphism

**Clojure Style Guide:**
- https://guide.clojure.style
- Comprehensive community-maintained style guide

**Apply this skill:**
- When writing new Clojure code
- When reviewing pull requests
- When refactoring existing code
- When explaining Clojure idioms to team members

Remember: These conventions exist to make code more readable, maintainable, and idiomatic. When in doubt, prioritize clarity and consistency with the existing codebase.
