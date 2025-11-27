# Malli Function Schemas Reference

Complete guide to function schemas and instrumentation in Malli.

## Overview

Malli provides powerful function schema capabilities for documenting, validating, and instrumenting function inputs and outputs. This enables runtime checking of function contracts and integration with static analysis tools.

## Basic Function Schemas

### Simple Function Types

```clojure
(require '[malli.core :as m])

; Function that takes an int and returns an int
[:=> [:cat :int] :int]

; Function that takes a string and int, returns a string
[:=> [:cat :string :int] :string]

; Function with multiple parameters
[:=> [:cat :string :int :boolean] :string]
```

### Shorthand Syntax

```clojure
; Using :-> for simpler syntax
[:-> :int :int]                      ; int → int
[:-> :string :int :string]           ; (string, int) → string
[:-> :int :int :int]                 ; (int, int) → int
```

### Defining Function Schemas

```clojure
(def add-schema
  [:=> [:cat :int :int] :int])

(def create-user-schema
  [:=>
   [:cat [:map [:email :string] [:name :string]]]
   [:map [:id uuid?] [:email :string] [:name :string]]])
```

## Associating Schemas with Functions

### Using m/=>

```clojure
(defn add [x y]
  (+ x y))

(m/=> add [:=> [:cat :int :int] :int])

; Now the function has an associated schema
(m/function-schema add)
;; => [:=> [:cat :int :int] :int]
```

### Inline Schema Definition

```clojure
(m/=> greet
  [:=> [:cat :string] :string])

(defn greet [name]
  (str "Hello, " name "!"))
```

### Multiple Functions

```clojure
(m/=> add [:=> [:cat :int :int] :int])
(m/=> subtract [:=> [:cat :int :int] :int])
(m/=> multiply [:=> [:cat :int :int] :int])
(m/=> divide [:=> [:cat :int :int] :double])
```

## Instrumentation

### Basic Instrumentation

```clojure
(require '[malli.instrument :as mi])

; Define function with schema
(m/=> add [:=> [:cat :int :int] :int])

(defn add [x y]
  (+ x y))

; Instrument all functions
(mi/instrument!)

; Now function calls are validated
(add 1 2)
;; => 3

(add "1" "2")
;; Throws validation error: invalid input
```

### Selective Instrumentation

```clojure
; Instrument specific function
(mi/instrument! {:filters [(mi/filter-ns 'myapp.core)]})

; Instrument by namespace
(mi/instrument! {:filters [(mi/filter-ns 'myapp.*)]})

; Instrument by function name
(mi/instrument! {:filters [(mi/filter-var #'add)]})
```

### Unstrument Functions

```clojure
; Unstrument all functions
(mi/unstrument!)

; Unstrument specific function
(mi/unstrument! {:filters [(mi/filter-var #'add)]})
```

### Development vs Production

```clojure
; Enable instrumentation only in development
(when (= :dev (config/environment))
  (mi/instrument!))

; Conditional instrumentation
(defn enable-instrumentation! []
  (when *assert*
    (mi/instrument!)))
```

## Multiple Arity Functions

### Function with Multiple Arities

```clojure
(def update-user-schema
  [:function
   [:=> [:cat uuid? [:map-of :keyword any?]] [:map]]
   [:=> [:cat uuid? [:map-of :keyword any?] [:map]] [:map]]])

(defn update-user
  ([user-id changes]
   (update-user user-id changes {}))
  ([user-id changes opts]
   {:user-id user-id
    :changes changes
    :opts opts}))

(m/=> update-user update-user-schema)
```

### Variadic Functions

```clojure
; Function accepting variable arguments
(def sum-schema
  [:=> [:cat [:* :int]] :int])

(defn sum [& numbers]
  (apply + numbers))

(m/=> sum sum-schema)

; Function with required and optional args
(def create-order-schema
  [:=>
   [:cat
    uuid?                             ; required: user-id
    [:vector :string]                 ; required: items
    [:* [:map-of :keyword any?]]]     ; optional: options
   [:map]])

(defn create-order [user-id items & opts]
  {:user-id user-id
   :items items
   :opts opts})

(m/=> create-order create-order-schema)
```

## Complex Function Schemas

### Nested Input/Output Schemas

```clojure
(def process-order-schema
  [:=>
   [:cat
    [:map
     [:order-id uuid?]
     [:items [:vector [:map
                       [:product-id uuid?]
                       [:quantity pos-int?]
                       [:price [:double {:min 0}]]]]]
     [:customer [:map
                 [:id uuid?]
                 [:email :string]
                 [:name :string]]]]]
   [:map
    [:order-id uuid?]
    [:status [:enum :pending :processing :completed]]
    [:total [:double {:min 0}]]]])

(defn process-order [order]
  {:order-id (:order-id order)
   :status :processing
   :total (reduce + (map #(* (:quantity %) (:price %)) (:items order)))})

(m/=> process-order process-order-schema)
```

### Function Returning Functions

```clojure
(def create-validator-schema
  [:=> [:cat any?] [:=> [:cat any?] :boolean]])

(defn create-validator [schema]
  (m/validator schema))

(m/=> create-validator create-validator-schema)
```

### Higher-Order Functions

```clojure
(def map-schema
  [:=>
   [:cat
    [:=> [:cat any?] any?]           ; mapper function
    [:sequential any?]]              ; input sequence
   [:sequential any?]])              ; output sequence

(m/=> map map-schema)
```

## Input Validation

### Cat (Concatenation)

Sequential parameters:

```clojure
[:=> [:cat :int :string :boolean] any?]

; Matches: (fn [n s b] ...)
```

### Catn (Named Cat)

Named parameters for better error messages:

```clojure
[:=> [:catn
      [:n :int]
      [:s :string]
      [:b :boolean]]
 any?]

; Better error messages reference parameter names
```

### Alt (Alternative)

Alternative parameter types:

```clojure
[:=> [:cat [:alt :int :string]] :string]

; Accepts either int or string as first parameter
```

### Altn (Named Alt)

```clojure
[:=> [:catn
      [:value [:altn
               [:num :int]
               [:text :string]]]]
 :string]
```

## Output Validation

### Simple Output

```clojure
[:=> [:cat :int :int] :int]          ; Returns int
[:=> [:cat :string] :boolean]        ; Returns boolean
[:=> [:cat any?] [:map]]             ; Returns map
```

### Complex Output

```clojure
[:=>
 [:cat uuid?]
 [:map
  [:id uuid?]
  [:created-at inst?]
  [:status [:enum :active :inactive]]]]
```

### Multiple Return Types

```clojure
[:=> [:cat :int] [:or :int :string]]

(defn int-or-string [n]
  (if (even? n)
    n
    (str n)))
```

## Function Properties

### Adding Metadata

```clojure
(def add-schema
  [:=> {:doc "Adds two integers"}
   [:cat :int :int]
   :int])

(m/=> add add-schema)
```

### Error Messages

```clojure
(def divide-schema
  [:=>
   [:cat
    :int
    [:int {:error/message "Divisor cannot be zero"} [:not [:= 0]]]]
   :double])

(defn divide [x y]
  (/ x y))

(m/=> divide divide-schema)
```

## Integration with Development Tools

### clj-kondo Integration

Malli function schemas can be exported for clj-kondo:

```clojure
(require '[malli.clj-kondo :as mc])

; Export schemas to .clj-kondo/config.edn
(mc/emit!)

; Now clj-kondo can use Malli schemas for static analysis
```

### REPL Development

```clojure
; Check function schema
(m/function-schema #'add)
;; => [:=> [:cat :int :int] :int]

; Validate manually
(m/validate [:=> [:cat :int :int] :int] add)
;; => true

; Test with sample data
(require '[malli.generator :as mg])

(let [inputs (mg/generate [:cat :int :int])]
  (apply add inputs))
```

## Testing Function Contracts

### Property-Based Testing

```clojure
(require '[clojure.test.check.properties :as prop]
         '[clojure.test.check.clojure-test :refer [defspec]]
         '[malli.generator :as mg])

(def add-schema
  [:=> [:cat :int :int] :int])

(m/=> add add-schema)

(defn add [x y]
  (+ x y))

; Test that function conforms to schema
(defspec add-conforms-to-schema 100
  (prop/for-all [inputs (mg/generator [:cat :int :int])]
    (let [[x y] inputs
          result (add x y)]
      (m/validate :int result))))

; Test specific properties
(defspec add-is-commutative 100
  (prop/for-all [inputs (mg/generator [:cat :int :int])]
    (let [[x y] inputs]
      (= (add x y) (add y x)))))
```

### Unit Testing with Schemas

```clojure
(require '[clojure.test :refer :all])

(deftest add-test
  (testing "Valid inputs"
    (is (= 3 (add 1 2)))
    (is (= 0 (add -1 1))))

  (testing "Function schema is correct"
    (is (m/function-schema #'add))))
```

## Advanced Patterns

### Preconditions and Postconditions

```clojure
(def safe-divide-schema
  [:=>
   [:catn
    [:x :int]
    [:y [:and :int [:not [:= 0]]]]]    ; y cannot be zero
   [:and :double [:not [:= ##Inf]]]])  ; result cannot be infinity

(defn safe-divide [x y]
  {:pre [(not= y 0)]}
  (double (/ x y)))

(m/=> safe-divide safe-divide-schema)
```

### State Transitions

```clojure
(def transition-schema
  [:=>
   [:cat
    [:map
     [:state [:enum :idle :running :stopped]]]
    [:enum :start :stop :reset]]
   [:map
    [:state [:enum :idle :running :stopped]]]])

(defn transition [machine action]
  (case [(:state machine) action]
    [:idle :start] (assoc machine :state :running)
    [:running :stop] (assoc machine :state :stopped)
    [:stopped :reset] (assoc machine :state :idle)
    machine))

(m/=> transition transition-schema)
```

### Dependent Return Types

```clojure
; Return type depends on input
(def parse-schema
  [:=>
   [:cat [:map [:format [:enum :json :edn]]]]
   any?])

(defn parse [opts]
  (case (:format opts)
    :json (json/parse-string (:input opts))
    :edn (edn/read-string (:input opts))))

(m/=> parse parse-schema)
```

## Registry Integration

### Function Schemas in Registry

```clojure
(def registry
  (merge
    (m/default-schemas)
    {::user
     [:map
      [:id uuid?]
      [:email :string]
      [:name :string]]

     ::create-user
     [:=> [:cat [:map [:email :string] [:name :string]]]
      ::user]

     ::update-user
     [:=> [:cat uuid? [:map-of :keyword any?]]
      ::user]}))

(m/=> create-user ::create-user {:registry registry})
(m/=> update-user ::update-user {:registry registry})
```

### Sharing Schemas

```clojure
(ns myapp.schemas
  (:require [malli.core :as m]))

(def User
  [:map
   [:id uuid?]
   [:email :string]
   [:name :string]])

(def CreateUserFn
  [:=> [:cat [:map [:email :string] [:name :string]]] User])

(def UpdateUserFn
  [:=> [:cat uuid? [:map-of :keyword any?]] User])

; Use in other namespaces
(ns myapp.users
  (:require [malli.core :as m]
            [myapp.schemas :as schemas]))

(m/=> create-user schemas/CreateUserFn)
(m/=> update-user schemas/UpdateUserFn)
```

## Best Practices

### 1. Use Descriptive Schemas

```clojure
; Good: Clear parameter names and types
(def create-user-schema
  [:=>
   [:catn
    [:email [:string {:min 1}]]
    [:name [:string {:min 1}]]
    [:age [:int {:min 0 :max 150}]]]
   [:map
    [:id uuid?]
    [:email :string]
    [:name :string]
    [:age :int]]])

; Less ideal: Generic cat without names
(def create-user-schema
  [:=> [:cat :string :string :int]
   [:map [:id uuid?] [:email :string] [:name :string] [:age :int]]])
```

### 2. Instrument During Development

```clojure
(defn enable-dev-instrumentation! []
  (when (= :dev (get-env :environment))
    (mi/instrument!)
    (println "Function instrumentation enabled")))
```

### 3. Document Function Contracts

```clojure
(def process-payment-schema
  [:=> {:doc "Processes a payment for an order"
        :added "1.2.0"}
   [:catn
    [:order-id {:doc "Order identifier"} uuid?]
    [:amount {:doc "Amount in cents"} pos-int?]
    [:payment-method {:doc "Payment method type"}
     [:enum :card :bank :paypal]]]
   [:map
    [:transaction-id uuid?]
    [:status [:enum :success :pending :failed]]
    [:timestamp inst?]]])
```

### 4. Test Function Schemas

```clojure
(deftest function-schema-test
  (testing "Schema is defined"
    (is (some? (m/function-schema #'my-function))))

  (testing "Generated inputs produce valid outputs"
    (let [input-gen (mg/generator (first (m/function-schema #'my-function)))
          inputs (mg/sample input-gen 10)]
      (doseq [input inputs]
        (is (m/validate (second (m/function-schema #'my-function))
                        (apply my-function input)))))))
```

### 5. Use with Guardrails

Combine Malli function schemas with Guardrails for even better development experience:

```clojure
(require '[guardrails.core :refer [>defn]])

(>defn create-user
  "Creates a new user"
  [email name age]
  [:=> [:cat :string :string [:int {:min 0 :max 150}]]
   [:map [:id uuid?] [:email :string] [:name :string] [:age :int]]]
  {:id (random-uuid)
   :email email
   :name name
   :age age})
```

## Common Use Cases

### API Handlers

```clojure
(def api-handler-schema
  [:=>
   [:cat [:map [:request any?]]]
   [:map
    [:status [:int {:min 100 :max 599}]]
    [:body any?]
    [:headers {:optional true} [:map-of :string :string]]]])

(m/=> handle-request api-handler-schema)
```

### Database Functions

```clojure
(def db-query-schema
  [:=>
   [:cat [:map [:query :string] [:params [:vector any?]]]]
   [:vector [:map-of :keyword any?]]])

(m/=> execute-query db-query-schema)
```

### Business Logic

```clojure
(def calculate-price-schema
  [:=>
   [:catn
    [:items [:vector [:map
                      [:product-id uuid?]
                      [:quantity pos-int?]
                      [:price [:double {:min 0}]]]]]
    [:discount {:optional true} [:double {:min 0 :max 1}]]]
   [:map
    [:subtotal [:double {:min 0}]]
    [:discount [:double {:min 0}]]
    [:total [:double {:min 0}]]]])

(m/=> calculate-price calculate-price-schema)
```
