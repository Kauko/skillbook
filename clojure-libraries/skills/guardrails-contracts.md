# Guardrails Runtime Contracts

Use when adding runtime contracts to Clojure functions. Uses Guardrails with Malli schemas and guardrails-analyzer for static checking at compile time.

## When to Use This Skill

- Adding runtime validation to function inputs and outputs
- Catching contract violations during development
- Documenting function contracts with executable specifications
- Setting up static analysis for contract checking
- Configuring production builds to remove contract overhead
- Integrating with Malli schemas for validation
- Establishing boundary validation at API layers

## Dependency Setup

### Check for Guardrails in deps.edn

```bash
grep -A 2 "com.fulcrologic/guardrails" deps.edn
```

### Add Dependencies if Missing

Add to `deps.edn`:

```clojure
{:deps {com.fulcrologic/guardrails {:mvn/version "1.1.10"}
        metosin/malli {:mvn/version "0.13.0"}}

 :aliases
 {:dev {:extra-deps {com.fulcrologic/guardrails {:mvn/version "1.1.10"}}}

  :build {:extra-deps {io.github.nubank/guardrails-analyzer
                       {:git/sha "01234abcdef"
                        :git/url "https://github.com/nubank/guardrails-analyzer"}}}}}
```

### Configuration Setup

Create `resources/config.edn` or update existing config:

```clojure
{:guardrails
 {:env :dev  ; :dev, :staging, or :production

  ;; Development configuration
  :dev {:enabled true
        :throw-on-error true
        :log-violations true}

  ;; Production configuration
  :production {:enabled false
               :compile-out true}}}
```

### Environment-Based Configuration

```clojure
(ns myapp.config
  (:require [guardrails.config :as gc]))

(defn init-guardrails! []
  (let [env (or (System/getenv "APP_ENV") "dev")]
    (case env
      "production" (gc/disable-guards!)
      "staging" (gc/enable-guards!)
      "dev" (gc/enable-guards!))))
```

## Basic Guardrails Usage

### Simple Function Contracts

```clojure
(ns myapp.core
  (:require [guardrails.core :refer [>defn =>]]))

;; Basic contract with primitive types
(>defn add
  [a b]
  [number? number? => number?]
  (+ a b))

;; Contract with string validation
(>defn greet
  [name]
  [string? => string?]
  (str "Hello, " name "!"))

;; Contract with multiple return types
(>defn divide
  [a b]
  [number? number? => [:or number? [:= ::division-by-zero]]]
  (if (zero? b)
    ::division-by-zero
    (/ a b)))
```

### Using Malli Schemas

```clojure
(ns myapp.domain.users
  (:require [guardrails.core :refer [>defn =>]]
            [malli.core :as m]))

;; Define Malli schema
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

;; Use schemas in contracts
(>defn create-user
  [request]
  [CreateUserRequest => User]
  (let [user (-> request
                 (select-keys [:email :name])
                 (assoc :id (random-uuid)))]
    user))

;; Multiple arguments with schemas
(>defn update-user
  [user-id changes]
  [uuid? [:map-of keyword? any?] => User]
  (merge (get-user user-id) changes))
```

### Contracts with Optional Arguments

```clojure
(>defn fetch-users
  ([query-params]
   [map? => [:vector User]]
   (fetch-users query-params {}))

  ([query-params options]
   [map? map? => [:vector User]]
   (let [limit (get options :limit 10)
         offset (get options :offset 0)]
     (db/query :users query-params {:limit limit :offset offset}))))
```

### Contracts with Varargs

```clojure
(>defn calculate-total
  [base & adjustments]
  [number? [:* number?] => number?]
  (reduce + base adjustments))

;; Usage
(calculate-total 100 10 -5 20)
;; => 125
```

## Advanced Patterns

### Registry-Based Schemas

```clojure
(ns myapp.schemas.registry
  (:require [malli.core :as m]))

(def registry
  (merge
    (m/default-schemas)
    {::user-id uuid?
     ::email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]

     ::user
     [:map
      [:id ::user-id]
      [:email ::email]
      [:name [:string {:min 1 :max 100}]]]

     ::order
     [:map
      [:id uuid?]
      [:user-id ::user-id]
      [:items [:vector ::order-item]]
      [:total [:double {:min 0}]]]

     ::order-item
     [:map
      [:product-id uuid?]
      [:quantity pos-int?]
      [:price [:double {:min 0}]]]}))

;; Use in contracts
(ns myapp.domain.orders
  (:require [guardrails.core :refer [>defn =>]]
            [myapp.schemas.registry :as schemas]))

(>defn process-order
  [order]
  [::schemas/order => ::schemas/order]
  ;; processing logic
  (assoc order :processed-at (java.time.Instant/now)))
```

### Pre and Post Conditions

```clojure
(>defn transfer-funds
  [from-account to-account amount]
  [::account ::account pos-int? => ::transaction-result]
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

### Higher-Order Functions

```clojure
(>defn map-users
  [f users]
  [[:=> [:cat ::user] any?] [:vector ::user] => [:vector any?]]
  (mapv f users))

(>defn filter-users
  [pred users]
  [[:=> [:cat ::user] boolean?] [:vector ::user] => [:vector ::user]]
  (filterv pred users))
```

### Async Functions

```clojure
(ns myapp.async
  (:require [guardrails.core :refer [>defn =>]]
            [clojure.core.async :as async]))

(>defn fetch-user-async
  [user-id]
  [uuid? => [:instance clojure.core.async.impl.channels.ManyToManyChannel]]
  (let [ch (async/chan)]
    (async/go
      (async/>! ch (fetch-user-from-db user-id)))
    ch))

;; Or with schema for channel contents (documented only)
(>defn fetch-user-async
  "Returns a channel that will contain a User"
  [user-id]
  [uuid? => any? #_"Channel<User>"]
  (let [ch (async/chan)]
    (async/go
      (async/>! ch (fetch-user-from-db user-id)))
    ch))
```

## Integration with Malli

### Shared Schema Definitions

Create schemas once, use everywhere:

```clojure
;; schemas/user.clj
(ns myapp.schemas.user
  (:require [malli.core :as m]))

(def User
  [:map
   [:id uuid?]
   [:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]
   [:name [:string {:min 1 :max 100}]]
   [:role [:enum :admin :user :guest]]])

(def CreateUserRequest
  [:map
   [:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]
   [:name [:string {:min 1 :max 100}]]
   [:password [:string {:min 8 :max 128}]]])

;; api/users.clj - Use for API validation
(ns myapp.api.users
  (:require [malli.core :as m]
            [myapp.schemas.user :as user-schemas]))

(defn create-user-handler [request]
  (let [body (:body request)]
    (if (m/validate user-schemas/CreateUserRequest body)
      {:status 200 :body (create-user body)}
      {:status 400 :body {:error "Invalid request"}})))

;; domain/users.clj - Use in Guardrails
(ns myapp.domain.users
  (:require [guardrails.core :refer [>defn =>]]
            [myapp.schemas.user :as user-schemas]))

(>defn create-user
  [request]
  [user-schemas/CreateUserRequest => user-schemas/User]
  (-> request
      (select-keys [:email :name])
      (assoc :id (random-uuid)
             :role :user)))
```

### Validation at Multiple Layers

```clojure
;; API Layer - Validate input
(defn create-user-handler [request]
  (let [body (:body request)]
    (if-let [errors (m/explain user-schemas/CreateUserRequest body)]
      {:status 400 :body {:errors (me/humanize errors)}}
      {:status 200 :body (domain/create-user body)})))

;; Domain Layer - Guardrails contract
(>defn create-user
  [request]
  [CreateUserRequest => User]
  (let [user (process-user-creation request)]
    ;; Contract validates output
    user))

;; Persistence Layer - Final validation before DB
(>defn save-user!
  [user]
  [User => User]
  (db/insert! :users user)
  user)
```

## Static Analysis with guardrails-analyzer

### Setup guardrails-analyzer

Add to `deps.edn`:

```clojure
{:aliases
 {:analyze {:extra-deps {io.github.nubank/guardrails-analyzer
                         {:git/sha "01234abcdef"
                          :git/url "https://github.com/nubank/guardrails-analyzer"}}
            :main-opts ["-m" "guardrails-analyzer.core"]}}}
```

### Run Static Analysis

```bash
# Analyze all source files
clojure -M:analyze src/

# Analyze specific namespace
clojure -M:analyze src/myapp/domain/users.clj

# Fail build on violations
clojure -M:analyze --fail-on-error src/
```

### Configuration

Create `.guardrails-analyzer.edn`:

```clojure
{:paths ["src"]
 :exclude ["src/myapp/generated"]
 :fail-on-error true
 :checks {:arity true
          :schema-validation true
          :return-type true}}
```

### CI/CD Integration

Add to your CI pipeline:

```yaml
# .github/workflows/ci.yml
- name: Run Guardrails Static Analysis
  run: clojure -M:analyze --fail-on-error src/
```

## clj-kondo Integration

### Setup clj-kondo Hooks

Create `.clj-kondo/hooks/guardrails.clj`:

```clojure
(ns hooks.guardrails
  (:require [clj-kondo.hooks-api :as api]))

(defn >defn [{:keys [node]}]
  (let [[fname & args] (rest (:children node))
        [args-vec schema & body] args
        defn-node (api/list-node
                    (list*
                      (api/token-node 'defn)
                      fname
                      args-vec
                      body))]
    {:node defn-node}))
```

### Configure clj-kondo

Add to `.clj-kondo/config.edn`:

```clojure
{:hooks {:analyze-call {guardrails.core/>defn hooks.guardrails/>defn}}
 :linters {:unresolved-symbol {:exclude [(guardrails.core/>defn)
                                         (guardrails.core/=>)]}}}
```

### Benefits

- clj-kondo will properly analyze `>defn` forms
- Arity checking works correctly
- Unused binding detection
- All standard clj-kondo linting applies

## Documentation Conventions

### Document Contracts in Vault

Create `vault/specifications/contracts.md`:

```markdown
# Runtime Contracts

This document describes our runtime contract conventions and patterns.

## Contract Principles

1. **Boundary Functions**: Always add contracts to public API functions
2. **Internal Functions**: Use contracts selectively for complex logic
3. **Performance**: Contracts are disabled in production builds
4. **Schemas**: Share Malli schemas between API validation and Guardrails

## Contract Locations

### API Layer
- All handler functions have contracts
- Validates HTTP request/response shapes
- Located in `myapp.api.*` namespaces

### Domain Layer
- Public domain functions have contracts
- Complex business logic validated
- Located in `myapp.domain.*` namespaces

### Persistence Layer
- Database write operations have contracts
- Ensures data integrity before persistence
- Located in `myapp.db.*` namespaces

## Examples

### User Management Contracts

**create-user**
- Input: `CreateUserRequest` schema
- Output: `User` schema
- Location: `myapp.domain.users/create-user`

**update-user**
- Input: `uuid?` user-id, `map?` changes
- Output: `User` schema
- Location: `myapp.domain.users/update-user`
```

### Inline Documentation

```clojure
(>defn process-payment
  "Processes a payment transaction with validation.

   Contract ensures:
   - Payment amount is positive
   - Payment method is valid
   - Customer exists
   - Returns completed transaction

   Throws: ExceptionInfo if payment fails validation"
  [payment customer-id]
  [::payment uuid? => ::transaction]
  ;; implementation
  )
```

## When to Use Contracts vs Boundary Validation

### Use Contracts When:

1. **Development Safety**: Catching bugs during development
2. **Documentation**: Contracts serve as executable documentation
3. **Internal APIs**: Functions called by other parts of your codebase
4. **Complex Logic**: Functions with intricate business rules

```clojure
;; Good use of contracts - internal domain function
(>defn calculate-discount
  [order customer]
  [::order ::customer => [:double {:min 0 :max 1}]]
  ;; Complex discount logic
  )
```

### Use Boundary Validation When:

1. **External Input**: HTTP requests, user input, external APIs
2. **Production Runtime**: Where performance matters
3. **Error Messages**: Need user-friendly error messages
4. **Always Enabled**: Validation that must run in production

```clojure
;; Good use of boundary validation - API handler
(defn create-order-handler [request]
  (let [order-data (get-in request [:body :order])]
    (if-let [errors (m/explain ::order order-data)]
      {:status 400
       :body {:errors (me/humanize errors)}}
      {:status 200
       :body (domain/create-order order-data)})))
```

### Layered Approach

```clojure
;; API Layer - Always validate (production)
(defn handler [request]
  (if-let [errors (validate-input request)]
    (error-response errors)
    (domain/process request)))

;; Domain Layer - Guardrails contract (dev only)
(>defn process
  [input]
  [::valid-input => ::result]
  ;; Business logic
  )
```

## Performance Considerations

### Contract Overhead

Contracts add runtime overhead during development:

```clojure
;; Without contract
(defn add [a b] (+ a b))
;; ~5ns per call

;; With contract
(>defn add [a b] [number? number? => number?] (+ a b))
;; ~500ns per call in dev (100x slower)
;; ~5ns per call in production (compiled out)
```

### Hot Path Warning

Avoid contracts in hot paths even during development:

```clojure
;; DON'T - called millions of times
(>defn calculate-pixel-color
  [r g b]
  [int? int? int? => [:map [:r int?] [:g int?] [:b int?]]]
  {:r r :g g :b b})

;; DO - validate at boundary instead
(defn render-image [pixels]
  (when-dev
    (assert (every? valid-pixel? pixels)))
  (doseq [p pixels]
    (calculate-pixel-color (:r p) (:g p) (:b p))))
```

### Selective Contracts

```clojure
;; Use contracts on public API
(>defn process-order
  [order]
  [::order => ::order-result]
  (let [validated (validate-order-items order)
        priced (calculate-pricing validated)]
    (finalize-order priced)))

;; Skip contracts on hot internal functions
(defn calculate-pricing [order]
  ;; No contract - called frequently
  (reduce calculate-item-price 0 (:items order)))

(defn calculate-item-price [total item]
  ;; No contract - called in tight loop
  (+ total (* (:quantity item) (:price item))))
```

## Production Configuration

### Compile Out Contracts

Configure shadow-cljs or ClojureScript build:

```clojure
;; shadow-cljs.edn
{:builds
 {:app
  {:target :browser
   :closure-defines {guardrails.config/ENABLED false}}}}
```

For Clojure:

```clojure
;; profiles.clj
{:prod {:jvm-opts ["-Dguardrails.enabled=false"]}}
```

### Environment Detection

```clojure
(ns myapp.config)

(def contracts-enabled?
  (if (= (System/getenv "APP_ENV") "production")
    false
    true))

;; Or read from config file
(def config (read-config "config.edn"))
(def contracts-enabled? (get-in config [:guardrails :enabled]))
```

### Verification

Verify contracts are disabled in production:

```clojure
(ns myapp.health
  (:require [guardrails.config :as gc]))

(defn health-check []
  {:status "ok"
   :guardrails {:enabled (gc/enabled?)}
   :environment (System/getenv "APP_ENV")})
```

## Common Patterns

### API Request/Response Pattern

```clojure
(ns myapp.api.orders
  (:require [guardrails.core :refer [>defn =>]]
            [myapp.schemas.api :as api-schemas]
            [myapp.schemas.order :as order-schemas]))

(>defn create-order-handler
  [request]
  [api-schemas/HttpRequest => api-schemas/HttpResponse]
  (let [order-data (get-in request [:body :order])]
    (if-let [result (domain/create-order order-data)]
      {:status 200 :body result}
      {:status 500 :body {:error "Failed to create order"}})))
```

### Domain Function Pattern

```clojure
(ns myapp.domain.orders
  (:require [guardrails.core :refer [>defn =>]]
            [myapp.schemas.order :as order-schemas]))

(>defn create-order
  [order-data]
  [order-schemas/CreateOrderRequest => order-schemas/Order]
  (-> order-data
      (validate-order-items)
      (calculate-totals)
      (save-order!)
      (publish-order-created-event)))

(>defn validate-order-items
  [order]
  [order-schemas/CreateOrderRequest => order-schemas/CreateOrderRequest]
  (if (seq (:items order))
    order
    (throw (ex-info "Order must have items" {:order order}))))
```

### Repository Pattern

```clojure
(ns myapp.db.users
  (:require [guardrails.core :refer [>defn =>]]
            [myapp.schemas.user :as user-schemas]))

(>defn save-user!
  [user]
  [user-schemas/User => user-schemas/User]
  (db/insert! :users user)
  user)

(>defn get-user
  [user-id]
  [uuid? => [:maybe user-schemas/User]]
  (db/get-by-id :users user-id))

(>defn list-users
  ([]
   [=> [:vector user-schemas/User]]
   (list-users {}))
  ([filters]
   [map? => [:vector user-schemas/User]]
   (db/query :users filters)))
```

### Error Handling Pattern

```clojure
(>defn process-with-fallback
  [data]
  [::input => [:or ::result ::error]]
  (try
    (process-data data)
    (catch Exception e
      {:error (.getMessage e)
       :type :processing-error})))

(>defn process-with-either
  [data]
  [::input => [:map
               [:success boolean?]
               [:result {:optional true} ::result]
               [:error {:optional true} string?]]]
  (try
    {:success true
     :result (process-data data)}
    (catch Exception e
      {:success false
       :error (.getMessage e)})))
```

## Testing with Contracts

### Contracts Aid Testing

```clojure
(ns myapp.domain.orders-test
  (:require [clojure.test :refer :all]
            [myapp.domain.orders :as orders]
            [malli.generator :as mg]
            [myapp.schemas.order :as order-schemas]))

;; Contracts catch bugs automatically
(deftest create-order-test
  (testing "Generated orders pass contracts"
    ;; If this fails, either:
    ;; 1. Generator is wrong
    ;; 2. Implementation is wrong
    ;; 3. Contract/schema is wrong
    (doseq [order (mg/sample order-schemas/CreateOrderRequest 100)]
      (is (orders/create-order order)))))
```

### Disable Contracts in Performance Tests

```clojure
(ns myapp.performance-test
  (:require [guardrails.config :as gc]))

(deftest performance-benchmark
  (gc/disable-guards!)
  (testing "Process 10000 orders in < 1 second"
    (time
      (doseq [order test-orders]
        (orders/process-order order))))
  (gc/enable-guards!))
```

## Migration Strategy

### Adding Contracts to Existing Code

1. Start with public API functions
2. Add to new code by default
3. Add to bug-prone areas
4. Gradually expand coverage

```clojure
;; Phase 1: Public API
(>defn create-user [data]
  [CreateUserRequest => User]
  ...)

;; Phase 2: Internal but complex
(>defn calculate-user-score [user activity]
  [User ActivityLog => number?]
  ...)

;; Phase 3: Simple internal functions (optional)
(>defn format-user-name [user]
  [User => string?]
  ...)
```

### Incremental Adoption

```clojure
;; Old function without contract
(defn legacy-function [a b]
  (+ a b))

;; Add contract alongside (don't break existing code)
(>defn legacy-function-v2 [a b]
  [number? number? => number?]
  (+ a b))

;; Gradually migrate callers
;; Eventually remove legacy-function
```

## Troubleshooting

### Contract Violation Errors

```clojure
;; Error: Input contract violation
;; Solution: Check your input data
(>defn process [x]
  [pos-int? => string?]
  (str x))

(process -1)  ; Violation!
(process 1)   ; OK
```

### Schema Not Found

```clojure
;; Error: Unable to resolve spec: ::user
;; Solution: Make sure schema is defined and required
(ns myapp.domain
  (:require [myapp.schemas.user :as user-schemas]))

(>defn process [user]
  [user-schemas/User => any?]  ; Use fully qualified
  ...)
```

### Performance Issues

```clojure
;; If contracts slow down development:
;; 1. Check if guardrails is only in :dev profile
;; 2. Temporarily disable for specific functions
;; 3. Use simpler schemas for hot paths

;; Disable for specific function
(defn hot-path-function [x]
  ;; No contract
  (expensive-calculation x))
```

## Best Practices Summary

1. **Always use contracts in :dev, disable in :prod**
2. **Share Malli schemas between API validation and Guardrails**
3. **Document contracts in vault/specifications/contracts.md**
4. **Use contracts for public APIs and boundaries**
5. **Skip contracts in hot paths**
6. **Set up guardrails-analyzer for static checking**
7. **Configure clj-kondo hooks for proper linting**
8. **Use boundary validation for user-facing errors**
9. **Generate tests from your schemas**
10. **Version your schemas as they evolve**

## References

- Guardrails: https://github.com/fulcrologic/guardrails
- guardrails-analyzer: https://github.com/nubank/guardrails-analyzer
- Malli integration: See `malli-schemas.md`
- clj-kondo hooks: https://github.com/clj-kondo/clj-kondo/tree/master/doc/hooks.md
