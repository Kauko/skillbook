# Guardrails Syntax Reference

This document provides comprehensive details on Guardrails syntax for defining runtime contracts.

## Core Syntax: Gspec

The fundamental syntax pattern for Guardrails contracts is called "gspec":

```
[arg-specs* (| arg-preds+)? => ret-spec (| ret-preds+)? (<- generator-fn)?]
```

### Components

- **arg-specs**: Type specifications for function arguments
- **`|`**: "Such that" separator for additional predicates
- **arg-preds**: Additional predicates that can reference argument symbols
- **`=>`**: Separates argument specs from return specs
- **ret-spec**: Type specification for return value
- **ret-preds**: Additional predicates on return value (using `%` for the result)
- **`<-`**: Optional generator function for property-based testing

### Basic Examples

```clojure
;; Simple types
[int? int? => int?]

;; With predicates
[int? int? | #(< start end) => int? | #(>= % start)]

;; Multiple arguments
[string? number? boolean? => map?]

;; No arguments
[=> string?]
```

## Primary Macros

### `>defn` - Defining Functions with Contracts

Replaces standard `defn` and adds inline validation. When disabled, emits normal `defn` without runtime overhead.

```clojure
(ns myapp.core
  (:require [guardrails.core :refer [>defn =>]]))

;; Basic usage
(>defn add
  [a b]
  [number? number? => number?]
  (+ a b))

;; With docstring
(>defn greet
  "Greets a person by name"
  [name]
  [string? => string?]
  (str "Hello, " name "!"))

;; Multiple arities
(>defn fetch-users
  ([query-params]
   [map? => [:vector User]]
   (fetch-users query-params {}))

  ([query-params options]
   [map? map? => [:vector User]]
   (db/query :users query-params options)))
```

### `>def` and `>fdef` - Spec Definitions

Define specs that can be removed from production builds:

```clojure
;; Define a spec
(>def ::user-id uuid?)

;; Function spec without implementation
(>fdef validate-user
  [user]
  [::user => boolean?])
```

### `?` - Nilable Shorthand

Shorthand for nullable/optional values:

```clojure
;; Without ?
[[:maybe string?] => [:maybe int?]]

;; With ?
[(? string?) => (? int?)]

;; Usage in function
(>defn find-user
  [email]
  [string? => (? User)]
  (db/find-one :users {:email email}))
```

## Such-That Predicates

Use `|` to add predicates that reference argument symbols:

```clojure
(>defn ranged-rand
  "Generate random int between start and end"
  [start end]
  [int? int? | #(< start end) => int? | #(>= % start) #(< % end)]
  (+ start (long (rand (- end start)))))

;; Multiple predicates
(>defn create-user
  [email age]
  [string? int? | #(pos? age) #(< age 150) => User]
  {:email email :age age :id (random-uuid)})

;; Return value predicates using %
(>defn calculate-discount
  [order]
  [Order => [:double | #(<= 0 % 1)]]
  (calculate-discount-logic order))
```

## Argument Specifications

### Simple Types

```clojure
;; Built-in predicates
[string? => boolean?]
[int? => double?]
[keyword? => symbol?]
[map? => vector?]
[coll? => seq?]

;; Combined types
[[:or string? keyword?] => string?]
[[:and sequential? [:not empty?]] => any?]
```

### Varargs

Use `[:*` for variable arguments:

```clojure
(>defn calculate-total
  [base & adjustments]
  [number? [:* number?] => number?]
  (reduce + base adjustments))

;; Usage
(calculate-total 100 10 -5 20) ;; => 125
```

### Maps and Collections

```clojure
;; Map with specific keys
[[:map
  [:id uuid?]
  [:name string?]
  [:age pos-int?]] => User]

;; Vector of items
[[:vector string?] => [:vector keyword?]]

;; Set
[[:set keyword?] => [:vector string?]]

;; Map-of (all keys/values same type)
[[:map-of keyword? string?] => map?]
```

### Optional Map Keys

```clojure
[[:map
  [:id uuid?]
  [:name string?]
  [:email {:optional true} string?]] => User]
```

### Higher-Order Functions

Specify function arguments:

```clojure
;; Function taking a function
(>defn map-users
  [f users]
  [[:=> [:cat User] any?] [:vector User] => [:vector any?]]
  (mapv f users))

;; Predicate function
(>defn filter-users
  [pred users]
  [[:=> [:cat User] boolean?] [:vector User] => [:vector User]]
  (filterv pred users))
```

## Return Value Specifications

### Simple Returns

```clojure
[string? => boolean?]
[User => User]
[int? int? => [:double]]
```

### Optional Returns

```clojure
;; May return nil
[uuid? => (? User)]
[string? => [:maybe User]]
```

### Union Types

```clojure
;; Multiple possible return types
[int? int? => [:or number? [:= ::division-by-zero]]]

;; Result or error
[map? => [:or ::success ::error]]
```

### Complex Return Types

```clojure
;; Detailed map structure
[CreateUserRequest => [:map
                       [:success boolean?]
                       [:user {:optional true} User]
                       [:error {:optional true} string?]]]

;; Nested collections
[query-params => [:vector [:map
                           [:id uuid?]
                           [:items [:vector Item]]]]]
```

### Async Return Types

```clojure
;; Promise/Future
[uuid? => [:instance java.util.concurrent.Future]]

;; Core.async channel
[uuid? => [:instance clojure.core.async.impl.channels.ManyToManyChannel]]

;; With documentation for channel contents
(>defn fetch-user-async
  "Returns a channel that will contain a User"
  [user-id]
  [uuid? => any? #_"Channel<User>"]
  (let [ch (async/chan)]
    (async/go
      (async/>! ch (fetch-user-from-db user-id)))
    ch))
```

## Malli Schema Integration

### Using Malli Schemas

```clojure
(require '[malli.core :as m])

;; Define schemas
(def User
  [:map
   [:id uuid?]
   [:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]
   [:name [:string {:min 1 :max 100}]]])

(def CreateUserRequest
  [:map
   [:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]
   [:name [:string {:min 1 :max 100}]]
   [:password [:string {:min 8}]]])

;; Use in contracts
(>defn create-user
  [request]
  [CreateUserRequest => User]
  (-> request
      (select-keys [:email :name])
      (assoc :id (random-uuid))))
```

### Schema Constraints

```clojure
;; String constraints
[[:string {:min 1 :max 100}] => string?]

;; Number constraints
[[:int {:min 0 :max 100}] => [:double {:min 0}]]

;; Collection constraints
[[:vector {:min 1 :max 10} string?] => [:vector keyword?]]

;; Regex patterns
[[:re #"^\d{3}-\d{3}-\d{4}$"] => boolean?]

;; Enum values
[[:enum :admin :user :guest] => User]
```

### Registry-Based Schemas

```clojure
;; Define registry
(def registry
  (merge
    (m/default-schemas)
    {::user-id uuid?
     ::email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]
     ::user [:map
             [:id ::user-id]
             [:email ::email]
             [:name [:string {:min 1 :max 100}]]]}))

;; Use qualified keywords
(>defn get-user
  [user-id]
  [::user-id => (? ::user)]
  (db/find-by-id :users user-id))
```

## Generators (Property-Based Testing)

### Specifying Generators

```clojure
;; Custom generator for testing
(>defn process-order
  [order]
  [Order => ProcessedOrder (<- order-generator)]
  (process-logic order))

;; Using with test.check
(require '[clojure.test.check.generators :as gen])

(>defn validate-email
  [email]
  [string? => boolean? (<- (gen/string-alphanumeric))]
  (re-matches #"^[^\s@]+@[^\s@]+\.[^\s@]+$" email))
```

## Advanced Patterns

### Pre and Post Conditions

Combine contracts with Clojure's pre/post conditions:

```clojure
(>defn transfer-funds
  [from-account to-account amount]
  [Account Account pos-int? => TransactionResult]
  {:pre [(>= (:balance from-account) amount)
         (not= (:id from-account) (:id to-account))]
   :post [(= (:amount %) amount)
          (= (:status %) :completed)]}
  (let [new-from (update from-account :balance - amount)
        new-to (update to-account :balance + amount)]
    {:from new-from
     :to new-to
     :amount amount
     :status :completed}))
```

### Error Types

```clojure
;; Define error types
(def ErrorTypes
  [:enum :not-found :invalid-input :server-error :unauthorized])

;; Use in return specs
(>defn fetch-user
  [user-id]
  [uuid? => [:or User [:tuple ErrorTypes string?]]]
  (try
    (db/get-user user-id)
    (catch Exception e
      [:server-error (.getMessage e)])))
```

### Conditional Return Types

```clojure
;; Either pattern
(>defn divide
  [a b]
  [number? number? => [:or number? [:= ::division-by-zero]]]
  (if (zero? b)
    ::division-by-zero
    (/ a b)))

;; Result map pattern
(>defn process-payment
  [payment]
  [Payment => [:map
               [:success boolean?]
               [:result {:optional true} ProcessedPayment]
               [:error {:optional true} string?]]]
  (try
    {:success true
     :result (charge-payment payment)}
    (catch Exception e
      {:success false
       :error (.getMessage e)})))
```

## Syntax Tips

### When to Use Such-That Predicates

Use `|` predicates when:
- Checking relationships between arguments
- Validating business rules beyond types
- Ensuring ordered/bounded values
- Cross-field validation

```clojure
;; Good: validates relationship
[int? int? | #(< start end) => int?]

;; Less useful: simple type check (use schema instead)
[int? | #(pos? x) => int?]  ;; Use pos-int? instead
```

### Keeping Contracts Concise

```clojure
;; Too complex - extract to schema
[[:map
  [:user [:map [:id uuid?] [:email string?]]]
  [:order [:map [:items [:vector [:map [:id uuid?]]]]]]
  => [:map [:success boolean?]]]

;; Better - use named schemas
[CheckoutRequest => CheckoutResult]
```

### Documentation Comments

```clojure
(>defn complex-function
  "Processes user data with validation.

  Contract ensures:
  - User has valid email format
  - Age is between 0 and 150
  - Returns either User or validation error

  Throws: ExceptionInfo on database errors"
  [email age]
  [[:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"] [:int | #(< 0 age 150)]
   => [:or User ValidationError]]
  ;; implementation
  )
```

## Common Patterns

### API Handler Pattern

```clojure
(def HttpRequest
  [:map
   [:method [:enum :get :post :put :delete]]
   [:path string?]
   [:body {:optional true} any?]])

(def HttpResponse
  [:map
   [:status [:int {:min 100 :max 599}]]
   [:body any?]])

(>defn handler
  [request]
  [HttpRequest => HttpResponse]
  {:status 200 :body {:message "success"}})
```

### Repository Pattern

```clojure
(>defn save!
  [entity]
  [Entity => Entity]
  (db/insert! entity))

(>defn find-by-id
  [id]
  [uuid? => (? Entity)]
  (db/get-by-id id))

(>defn list-all
  ([]
   [=> [:vector Entity]]
   (list-all {}))
  ([filters]
   [map? => [:vector Entity]]
   (db/query filters)))
```

### Domain Function Pattern

```clojure
(>defn create-order
  [order-data]
  [CreateOrderRequest => Order]
  (-> order-data
      validate-items
      calculate-totals
      save-order!
      publish-event))
```

## References

- Guardrails GitHub: https://github.com/fulcrologic/guardrails
- Malli Documentation: https://github.com/metosin/malli
- Clojure Spec Guide: https://clojure.org/guides/spec
