# Clojure Style Guide Reference

Comprehensive reference from the community-maintained Clojure Style Guide. These conventions represent community consensus on idiomatic Clojure code.

**Source:** https://guide.clojure.style

---

## Naming Conventions

### Functions and Variables: lisp-case (kebab-case)

```clojure
;; GOOD
(def some-var 42)
(defn calculate-sum [a b]
  (+ a b))

;; BAD
(def someVar 42)           ; camelCase
(def some_var 42)          ; snake_case
(defn CalculateSum [a b]   ; PascalCase
  (+ a b))
```

**Rationale:** Kebab-case is the universal Clojure standard. It distinguishes Clojure code from Java interop and improves readability.

---

### Predicates: End with `?`

```clojure
;; GOOD
(defn palindrome? [s]
  (= s (clojure.string/reverse s)))

(defn valid-email? [s]
  (re-matches #".+@.+\..+" s))

(when (empty? coll)
  ...)

;; BAD
(defn palindrome [s]          ; Missing ?
  ...)

(defn is-palindrome [s]       ; Don't use is- prefix
  ...)

(defn palindrome-p [s]        ; Common Lisp convention, not Clojure
  ...)
```

**Rationale:** The `?` suffix immediately signals a boolean return value. Avoid verbose prefixes like `is-`, `has-`, or Common Lisp's `-p` suffix.

---

### Unsafe Operations: End with `!`

```clojure
;; GOOD - mutating operations
(swap! atom-state update :counter inc)
(reset! atom-state {})
(send! agent-ref process)

;; GOOD - I/O and side effects
(defn save-to-db! [data]
  (jdbc/insert! db :users data))

(defn send-email! [to subject body]
  (mail/send {...}))

;; GOOD - breaking encapsulation
(defn set-field! [obj field value]
  (set! (.-field obj) value))
```

**Rationale:** The `!` warns readers that:
- The function mutates state (dangerous in STM contexts)
- Side effects occur (I/O, logging, etc.)
- Encapsulation boundaries are crossed
- Extra caution is needed

**Note:** Not all side-effecting functions need `!`. It's primarily for:
- Direct mutation (atoms, refs, agents, transients)
- Unsafe operations that could cause issues in concurrent contexts
- Clear warning signals in critical code paths

---

### Dynamic Vars: Wrap with `*earmuffs*`

```clojure
;; GOOD
(def ^:dynamic *db-connection* nil)
(def ^:dynamic *current-user* nil)
(def ^:dynamic *max-retries* 3)

(binding [*db-connection* conn
          *current-user* user]
  (perform-operation))

;; BAD
(def ^:dynamic db-connection nil)      ; Missing earmuffs
(def ^:dynamic current-user nil)
```

**Rationale:** Earmuffs make dynamic vars immediately recognizable:
- Signals thread-local binding semantics
- Prevents confusion with regular vars
- Warns about potential binding context requirements
- Long-standing Lisp tradition

---

### Types, Records, and Protocols: CapitalCase

```clojure
;; GOOD
(defprotocol DataStore
  (save [this data])
  (load [this id])
  (delete [this id]))

(defrecord User [id name email])

(deftype CustomQueue [items])

(definterface ICustom
  (method []))

;; BAD
(defprotocol data-store        ; Should be CapitalCase
  ...)

(defrecord user [id name])     ; Should be User

(deftype custom-queue [items]) ; Should be CustomQueue
```

**Rationale:**
- CapitalCase distinguishes types from functions
- Aligns with Java conventions for interop
- Makes constructors like `->User` obvious
- Follows object-oriented language conventions

---

### Unused Bindings: Use `_`

```clojure
;; GOOD - single underscore for unused bindings
(let [[first-item _ third-item] collection]
  (process first-item third-item))

(fn [_ event]  ; Ignore first argument
  (handle-event event))

(doseq [[k _] my-map]  ; Only need keys
  (println k))

;; GOOD - prefixed underscore when name aids understanding
(let [[first-item _second-item third-item] collection]
  (process first-item third-item))

;; BAD - named but unused
(let [[first-item second-item third-item] collection]
  (process first-item third-item))  ; second-item unused
```

**Rationale:**
- `_` signals intentional ignoring
- Prevents unused variable warnings
- Documents intent clearly
- Prefix with `_` when the name aids comprehension

---

### Namespaces: Multi-Segment Names

```clojure
;; GOOD
(ns myapp.orders.core)
(ns mycompany.myapp.auth)
(ns acme.web.routes)

;; BAD
(ns orders)   ; Single segment - collision risk
(ns core)     ; Too generic
(ns utils)    ; Meaningless without context
```

**Rationale:**
- Multi-segment names avoid collisions
- Provide context and organization
- Single-segment names reserved for Clojure core and established libraries
- Typical pattern: `company.project.module` or `project.module`

---

### Private Functions: Mark with `^:private` or `defn-`

```clojure
;; GOOD
(defn- private-helper [x]
  (* x 2))

(def ^:private private-var 42)

;; GOOD - public API
(defn public-function [x]
  (private-helper x))

;; BAD - helper should be private
(defn helper [x]  ; Exposed in public API
  (* x 2))
```

**Rationale:**
- Clearly separates public API from implementation
- Prevents external dependencies on internal functions
- Enables refactoring without breaking changes

---

## Code Organization

### One Namespace Per File

```clojure
;; GOOD: src/myapp/orders/core.clj
(ns myapp.orders.core
  (:require [myapp.db :as db]))

;; BAD: Don't put multiple namespaces in one file
(ns myapp.orders.core ...)
(ns myapp.orders.utils ...)  ; Should be separate file
```

**Rationale:** One-to-one mapping between files and namespaces makes code easy to locate and navigate.

---

### Namespace Declaration Order

```clojure
;; GOOD - standard order
(ns myapp.core
  "Namespace docstring explaining purpose."
  (:refer-clojure :exclude [get])
  (:require [clojure.string :as str]
            [clojure.set :as set]
            [myapp.config :as config]
            [myapp.db :as db])
  (:import [java.time Instant LocalDate]
           [java.util UUID Date]))

;; BAD - wrong order
(ns myapp.core
  (:import [java.util UUID])
  (:require [clojure.string :as str]))  ; Should be before :import
```

**Standard order:**
1. Docstring (optional)
2. `:refer-clojure` (if needed)
3. `:require`
4. `:import`
5. `:gen-class` (rarely needed)

**Rationale:** Consistent ordering improves scannability and follows community convention.

---

### Require Aliases

```clojure
;; GOOD - clear, standard aliases
(ns myapp.core
  (:require [clojure.string :as str]
            [clojure.set :as set]
            [clojure.java.io :as io]
            [myapp.database :as db]
            [myapp.users :as users]))

;; GOOD - full namespace for clarity
(myapp.utils/format-date date)

;; BAD - :refer :all pollutes namespace
(ns myapp.core
  (:require [myapp.utils :refer :all]))

;; BAD - too short, unclear
(ns myapp.core
  (:require [myapp.database :as d]))
```

**Guidelines:**
- Use standard aliases: `str`, `set`, `io`
- Make aliases meaningful and readable
- Avoid `:refer :all` (exception: test namespaces with `clojure.test`)
- Prefer qualified names for clarity

---

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

**Rationale:** Single blank lines improve readability without excessive whitespace.

---

### No Internal Blank Lines in Definitions

```clojure
;; GOOD - no internal blank lines
(defn process-order
  [order]
  (let [total (calculate-total order)
        discount (calculate-discount order)]
    (if (valid? total)
      (save-order! order total)
      (throw (ex-info "Invalid total" {:total total})))))

;; BAD - unnecessary blank lines inside function
(defn process-order
  [order]

  (let [total (calculate-total order)
        discount (calculate-discount order)]

    (if (valid? total)
      (save-order! order total)

      (throw (ex-info "Invalid total" {:total total})))))
```

**Rationale:**
- Blank lines should separate top-level forms
- Internal blank lines clutter function bodies
- Exception: Complex functions may use blank lines to separate logical sections

---

## Formatting

### Line Length: 80-120 Characters

```clojure
;; GOOD - within 80 characters (ideal)
(defn process-payment [order payment-method]
  ...)

;; ACCEPTABLE - up to 120 for readability
(defn calculate-shipping [order destination-address shipping-method]
  ...)

;; BAD - too long, should be split
(defn calculate-total-with-discounts-and-taxes [order customer-level coupon-codes tax-rates]
  ...)
```

**Guideline:** 80 characters ideal, 120 acceptable. Configure your editor accordingly.

---

### Indentation: 2 Spaces, Never Tabs

```clojure
;; GOOD - consistent 2-space indentation
(defn my-function
  [arg]
  (let [x 1
        y 2]
    (+ x y arg)))

;; BAD - inconsistent or tab-based indentation
(defn my-function
[arg]
    (let [x 1
      y 2]
  (+ x y arg)))
```

**Rationale:**
- 2 spaces is universal in Clojure
- Tabs cause alignment issues
- Use cljfmt or similar tools to enforce

---

### Semantic Indentation

```clojure
;; GOOD - forms align semantically
(when (valid? data)
  (process data)
  (save data))

(let [users (fetch-users)
      orders (fetch-orders users)
      reports (generate-reports users orders)]
  (save-reports! reports))

;; GOOD - vertically align function arguments
(filter even?
        (range 1 10))

(defn long-function-name
  [first-arg
   second-arg
   third-arg]
  ...)

;; BAD - inconsistent alignment
(when (valid? data)
(process data)
(save data))
```

**Rationale:** Semantic indentation reveals code structure at a glance.

---

### Trailing Parentheses: Gather on Single Line

```clojure
;; GOOD
(when something
  (something-else))

(defn foo
  [x]
  (let [y (+ x 1)]
    (println y)))

;; BAD - scattered closing parens
(when something
  (something-else)
  )

(defn foo
  [x]
  (let [y (+ x 1)]
    (println y)
    )
  )
```

**Rationale:**
- Trailing parens on same line as code
- Reduces visual clutter
- Follows Lisp convention
- Use parinfer or paredit tools

---

### Bracket Spacing

```clojure
;; GOOD - space before opening, after closing brackets
(foo (bar baz) quux)
[1 2 3]
{:a 1 :b 2}

;; BAD - no spacing
(foo(bar baz)quux)

;; BAD - space after opening brackets
( foo (bar baz) quux )
[ 1 2 3 ]
{ :a 1 :b 2 }
```

---

### No Commas in Sequential Collections

```clojure
;; GOOD
[1 2 3 4]
'(1 2 3 4)
#{1 2 3 4}

;; BAD - unnecessary commas
[1, 2, 3, 4]

;; EXCEPTION - commas in maps for readability
{:name "Alice"
 :age 30}  ; No comma after :name

;; ACCEPTABLE - commas in maps for pair clarity
{:name "Alice", :age 30, :email "alice@example.com"}
```

**Rationale:** Commas are whitespace in Clojure. Use them sparingly in maps for clarity.

---

## Idiomatic Patterns

### Threading Macros: Reduce Nesting

```clojure
;; GOOD - threading shows data flow
(-> data
    (assoc :id (UUID/randomUUID))
    (update :timestamp #(or % (Instant/now)))
    (dissoc :internal-field))

(->> users
     (filter :active?)
     (map :email)
     (take 10))

;; BAD - deeply nested
(dissoc
  (update
    (assoc data :id (UUID/randomUUID))
    :timestamp #(or % (Instant/now)))
  :internal-field)

(->> users
     (filter :active?)
     (map :email)
     (reduce (fn [acc email]
               (if (valid? email)
                 (conj acc email)
                 acc))
             [])
     (take 10))
```

**Guidelines:**
- `->` threads as first argument (for maps, objects)
- `->>` threads as last argument (for sequences)
- `as->` for mixed positions (within threading chain only)
- `cond->` for conditional threading
- `some->` for nil-short-circuiting

---

### Use `when` for Single-Branch Conditionals

```clojure
;; GOOD - when has implicit do
(when (logged-in? user)
  (log-access user)
  (fetch-data user))

;; GOOD - when-not for negative conditions
(when-not (cached? key)
  (fetch-from-db key)
  (cache! key))

;; BAD - unnecessary if with do
(if (logged-in? user)
  (do
    (log-access user)
    (fetch-data user))
  nil)

;; BAD - single-branch if
(if (logged-in? user)
  (do
    (log-access user)
    (fetch-data user)))
```

---

### Use `seq` for Empty Collection Tests

```clojure
;; GOOD - seq returns nil for empty
(when (seq collection)
  (process collection))

(if (seq items)
  (first items)
  default-value)

;; BAD - double negative
(when (not (empty? collection))
  (process collection))

;; BAD - count is inefficient
(when (> (count items) 0)
  (process items))

;; BAD - even worse for lazy sequences
(when (not= 0 (count lazy-seq))
  ...)  ; Realizes entire sequence!
```

**Rationale:**
- `seq` returns nil for empty, enabling idiomatic nil-punning
- More efficient than `count` (especially for lazy sequences)
- More idiomatic than `empty?`

---

### Combine Binding with Conditionals

```clojure
;; GOOD - if-let combines binding and test
(if-let [user (find-user id)]
  (greet user)
  (create-guest))

;; GOOD - when-let for single branch
(when-let [config (load-config path)]
  (validate config)
  (apply-config config))

;; GOOD - if-some when nil is valid value
(if-some [result (compute x)]
  result
  :not-found)

;; BAD - unnecessary two-step
(let [user (find-user id)]
  (if user
    (greet user)
    (create-guest)))
```

**Note:** `if-let` and `when-let` test for truthiness, not just nil.

---

### Prefer Higher-Order Functions

```clojure
;; GOOD - declarative, clear intent
(map #(* % 2) numbers)
(filter even? numbers)
(reduce + 0 numbers)
(->> data
     (group-by :category)
     (map (fn [[k v]] [k (count v)])))

;; BAD - unnecessary loop
(loop [nums numbers
       result []]
  (if (seq nums)
    (recur (rest nums) (conj result (* 2 (first nums))))
    result))
```

**When to use `loop/recur`:**
- Performance-critical code requiring tail recursion
- Complex iteration that doesn't fit HOF patterns
- State machines or stateful iteration

---

### Prefer `condp` for Repeated Predicates

```clojure
;; GOOD - condp eliminates repetition
(condp = status
  :pending (handle-pending order)
  :approved (handle-approved order)
  :rejected (handle-rejected order)
  :cancelled (handle-cancelled order)
  (handle-unknown order))

;; GOOD - condp with different predicates
(condp some [status]
  #{:pending :processing} (handle-active order)
  #{:completed :shipped} (handle-finished order)
  (handle-other order))

;; BAD - repetitive cond
(cond
  (= status :pending) (handle-pending order)
  (= status :approved) (handle-approved order)
  (= status :rejected) (handle-rejected order)
  (= status :cancelled) (handle-cancelled order)
  :else (handle-unknown order))
```

---

### Collections as Functions

```clojure
;; GOOD - idiomatic Clojure
(:name user)
(user :name)
(#{:admin :moderator} role)
([1 2 3] 1)  ; => 2

;; ACCEPTABLE - explicit but verbose
(get user :name)
(contains? #{:admin :moderator} role)
(nth [1 2 3] 1)
```

**Note:** While idiomatic, remember to use explicit fallbacks with `get` (see custom Metosin convention).

---

### Function Parameters: Limit to 3-4

```clojure
;; GOOD - few parameters
(defn create-user [name email]
  ...)

;; ACCEPTABLE - map for many parameters
(defn create-order
  [{:keys [user-id items shipping-address payment-method discount-code]}]
  ...)

;; BAD - too many positional parameters
(defn create-order [user-id item1 item2 item3 shipping-address payment-method discount-code]
  ...)
```

**Rationale:**
- Beyond 3-4 parameters, use a map
- Maps provide named parameters
- Order doesn't matter with maps
- Easier to add optional parameters

---

### Function Length: Aim for Under 10 Lines

```clojure
;; GOOD - focused, single responsibility
(defn valid-email? [s]
  (and (string? s)
       (re-matches #".+@.+\..+" s)))

(defn process-order [order]
  (let [validated (validate-order order)
        priced (calculate-pricing validated)
        saved (save-order! priced)]
    saved))

;; BAD - too long, multiple responsibilities
(defn process-order [order]
  ;; 50+ lines of validation, pricing, inventory check,
  ;; payment processing, notification, logging...
  ...)
```

**Guideline:** Functions over 10-15 lines often indicate:
- Multiple responsibilities (split into separate functions)
- Complex logic (extract helper functions)
- Missing abstractions

---

### Use Pre/Post Conditions

```clojure
;; GOOD - pre/post conditions for contracts
(defn divide [numerator denominator]
  {:pre [(number? numerator)
         (number? denominator)
         (not= 0 denominator)]
   :post [(number? %)]}
  (/ numerator denominator))

(defn process-adult-user [user]
  {:pre [(:age user)
         (>= (:age user) 18)]}
  (grant-full-access user))
```

**Rationale:**
- Documents expectations and invariants
- Fails fast with clear error messages
- Acts as executable documentation

---

## Documentation

### Docstrings for Public Functions

```clojure
;; GOOD - comprehensive docstring
(defn calculate-discount
  "Calculates discount amount for an order based on customer type and coupon.

  Args:
    order  - Map with :total (number) and :customer-type (keyword)
    coupon - Optional coupon code string

  Returns:
    Discount amount as number (0 to order total)

  Example:
    (calculate-discount {:total 100 :customer-type :premium} \"SAVE20\")
    ;=> 20.0"
  [order coupon]
  ...)

;; ACCEPTABLE - brief docstring
(defn calculate-discount
  "Returns discount amount for order and coupon code."
  [order coupon]
  ...)

;; BAD - missing docstring for public function
(defn calculate-discount [order coupon]
  ...)
```

**Include:**
- Purpose and behavior
- Parameter descriptions
- Return value
- Examples (for complex functions)
- Side effects and exceptions

---

### Comment Hierarchy

```clojure
;;;; Major Section: Order Processing
;;;; This section handles the complete order lifecycle from
;;;; validation through fulfillment.

;;; Subsection: Validation
;;; Functions for validating orders before processing.

;; Function-level comment
;; This function handles the special case where...
(defn validate-special-order [order]
  (let [rules (get-rules order)]
    ;; Apply rules with custom logic
    (apply-rules rules order)))  ; Inline comment
```

**Comment levels:**
- `;;;;` - Major section headings
- `;;;` - Subsection descriptions
- `;;` - Function-level comments (above function)
- `;` - Inline comments (end of line)

---

### Prioritize Self-Explanatory Code

```clojure
;; GOOD - code explains itself
(defn active-premium-users [users]
  (->> users
       (filter :active?)
       (filter #(= :premium (:tier %)))))

;; BAD - comments explain unclear code
(defn get-users [users]
  ;; Filter for active users
  ;; Then filter for premium tier only
  (->> users
       (filter :active?)
       (filter #(= :premium (:tier %)))))
```

**Rationale:** Use comments for "why", not "what". Clear names eliminate most comment needs.

---

## Error Handling

### Use `ex-info` for Rich Exceptions

```clojure
;; GOOD - structured error data
(when-not (valid? order)
  (throw (ex-info "Invalid order"
                  {:order order
                   :errors (validate order)
                   :type ::validation-error})))

(when (< balance amount)
  (throw (ex-info "Insufficient funds"
                  {:balance balance
                   :required amount
                   :shortfall (- amount balance)
                   :type ::insufficient-funds})))

;; BAD - string-only exception
(when-not (valid? order)
  (throw (Exception. "Invalid order")))
```

**Benefits:**
- Structured data enables programmatic error handling
- Better debugging with context
- Can catch and inspect by type
- Preserves information

---

### Descriptive Error Messages

```clojure
;; GOOD - specific, actionable
(when-not (contains? order :customer-id)
  (throw (ex-info "Order missing required field :customer-id"
                  {:order order
                   :required-fields [:customer-id :items :total]})))

;; GOOD - explains what went wrong
(when (< (count items) min-items)
  (throw (ex-info (str "Order requires at least " min-items " items")
                  {:items (count items)
                   :minimum min-items})))

;; BAD - vague
(throw (ex-info "Invalid order" {:order order}))

;; BAD - no context
(throw (Exception. "Error"))
```

---

## Testing

### Descriptive Test Names

```clojure
;; GOOD - describes behavior being tested
(deftest calculate-discount-applies-percentage-for-valid-coupon
  (is (= 20.0 (calculate-discount {:total 100} "SAVE20"))))

(deftest calculate-discount-returns-zero-for-invalid-coupon
  (is (= 0.0 (calculate-discount {:total 100} "INVALID"))))

(deftest calculate-discount-caps-at-order-total
  (is (= 100.0 (calculate-discount {:total 100} "SAVE200"))))

;; BAD - generic names
(deftest test1
  (is (= 20.0 (calculate-discount {:total 100} "SAVE20"))))

(deftest discount-test
  (is (= 20.0 (calculate-discount {:total 100} "SAVE20"))))
```

---

### Use `testing` for Context

```clojure
;; GOOD - organized with context
(deftest order-validation
  (testing "valid orders"
    (is (valid-order? {:customer-id 1 :items [{:id 1}]})))

  (testing "missing customer-id"
    (is (not (valid-order? {:items [{:id 1}]}))))

  (testing "missing items"
    (is (not (valid-order? {:customer-id 1}))))

  (testing "empty items"
    (is (not (valid-order? {:customer-id 1 :items []})))))
```

**Benefits:**
- Groups related assertions
- Provides context in failure messages
- Organizes test structure

---

## Performance

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

**When to use:**
- Building large collections in a loop
- Many updates to same collection
- Performance-critical code

**Rules:**
- Create with `transient`
- Modify with `assoc!`, `conj!`, `dissoc!`
- Finalize with `persistent!`
- Never leak transients outside function

---

### Type Hints for Performance

```clojure
;; GOOD - type hints prevent reflection
(defn process-string [^String s]
  (.toUpperCase s))

(defn calculate [^long x ^long y]
  (+ x y))

;; Check for reflection warnings
(set! *warn-on-reflection* true)
```

**When to use:**
- Interop with Java
- Performance-critical code
- After profiling shows reflection overhead

---

### Don't Shadow Core Names

```clojure
;; BAD - shadows clojure.core
(defn map [coll]
  ...)

(let [filter (fn [x] ...)]
  ...)

;; GOOD - different names
(defn transform [coll]
  ...)

(let [pred (fn [x] ...)]
  ...)
```

**Common shadows to avoid:**
`map`, `filter`, `remove`, `range`, `set`, `list`, `name`, `key`, `val`, `get`, `update`, `merge`, `replace`, `type`, `load`

---

## Summary: Quick Reference

**Naming:**
- Functions/vars: kebab-case
- Predicates: `name?`
- Unsafe ops: `name!`
- Dynamic vars: `*name*`
- Types: CapitalCase
- Unused: `_` or `_name`

**Organization:**
- One namespace per file
- Multi-segment namespace names
- Standard ns order: docstring, :require, :import
- Single blank line between top-level forms

**Formatting:**
- 80-120 character lines
- 2-space indentation, never tabs
- Semantic indentation
- Trailing parens on same line
- No commas in sequences

**Idioms:**
- Use threading macros
- `when` for single-branch
- `seq` for empty tests
- `if-let`/`when-let` for binding
- Higher-order functions over `loop/recur`
- `condp` for repeated predicates
- Limit parameters to 3-4

**Documentation:**
- Docstrings for public functions
- Comment hierarchy: `;;;;` `;;;` `;;` `;`
- Self-explanatory code over comments

**Error Handling:**
- `ex-info` with structured data
- Descriptive, specific messages

**Testing:**
- Descriptive test names
- Use `testing` for context

**Additional Resources:**
- Full guide: https://guide.clojure.style
- cljfmt for automatic formatting
- clj-kondo for linting
- parinfer/paredit for structural editing
