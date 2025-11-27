# Malli Schema Patterns

Use when working with data validation, schemas, or generative testing in Clojure. Guides Malli usage patterns for robust data validation and transformation.

## When to Use This Skill

- Defining schemas for domain entities and data structures
- Validating API requests and responses
- Setting up configuration validation
- Creating generative tests with property-based testing
- Documenting data shapes and contracts
- Transforming and coercing data
- Building registry-based schema systems

## Reference Documentation

For comprehensive details, see:
- `references/schema-types.md` - Complete reference of all built-in schema types
- `references/transformations.md` - Data transformation, coercion, encoding/decoding
- `references/generation.md` - Generative testing and property-based testing
- `references/function-schemas.md` - Function schemas and instrumentation

## Dependency Setup

### Check for Malli in deps.edn

First, verify if Malli is already present:

```bash
grep -A 2 "metosin/malli" deps.edn
```

### Add Malli if Missing

If not present, add to `deps.edn`:

```clojure
{:deps {metosin/malli {:mvn/version "0.13.0"}}}
```

For development and testing, also add:

```clojure
{:aliases
 {:dev {:extra-deps {metosin/malli {:mvn/version "0.13.0"}}}
  :test {:extra-deps {org.clojure/test.check {:mvn/version "1.1.1"}}}}}
```

## Core Malli Concepts

### Basic Schema Types

Malli schemas are first-class data structures that describe the shape of your data.

```clojure
(require '[malli.core :as m])

;; Primitive types
int?             ;; Integer
string?          ;; String
boolean?         ;; Boolean
keyword?         ;; Keyword
uuid?            ;; UUID
inst?            ;; Instance/Date

;; Numeric constraints
pos-int?         ;; Positive integer
nat-int?         ;; Natural number (>= 0)
neg-int?         ;; Negative integer
double?          ;; Double
number?          ;; Any number

;; String patterns
:string          ;; Any string
[:string {:min 1, :max 100}]  ;; Length constraints
[:re #"^[a-z]+$"]             ;; Regex validation

;; Collections
[:vector any?]                ;; Vector of anything
[:vector int?]                ;; Vector of integers
[:set keyword?]               ;; Set of keywords
[:map                         ;; Map with specific keys
 [:id uuid?]
 [:name string?]]

;; Optional and nullable
[:maybe string?]              ;; String or nil
[:map
 [:id uuid?]
 [:name string?]
 [:email {:optional true} string?]]  ;; Optional key
```

**Note**: Maps are open by default (allow extra keys). Use `{:closed true}` to restrict to declared keys only. See `references/schema-types.md` for complete type reference.

## Schema Design Patterns

### 1. Entity Schemas

Define schemas for your domain entities:

```clojure
(ns myapp.schemas.user
  (:require [malli.core :as m]
            [malli.util :as mu]))

;; Simple entity
(def User
  [:map
   [:id uuid?]
   [:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]
   [:name [:string {:min 1 :max 100}]]
   [:age [:int {:min 0 :max 150}]]
   [:created-at inst?]])

;; Nested entity
(def Address
  [:map
   [:street string?]
   [:city string?]
   [:postal-code [:string {:min 5 :max 10}]]
   [:country [:enum "FI" "SE" "NO" "DK"]]])

(def UserWithAddress
  [:map
   [:id uuid?]
   [:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]
   [:name string?]
   [:address Address]])

;; Entity with optional fields
(def UserProfile
  [:map
   [:id uuid?]
   [:email string?]
   [:name string?]
   [:bio {:optional true} [:string {:max 500}]]
   [:avatar-url {:optional true} string?]
   [:preferences {:optional true}
    [:map
     [:theme [:enum "light" "dark"]]
     [:language [:enum "en" "fi" "sv"]]]]])
```

### 2. API Request/Response Schemas

```clojure
(ns myapp.schemas.api
  (:require [malli.core :as m]))

;; Request schemas
(def CreateUserRequest
  [:map
   [:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]
   [:name [:string {:min 1 :max 100}]]
   [:password [:string {:min 8 :max 128}]]])

(def UpdateUserRequest
  [:map
   [:name {:optional true} [:string {:min 1 :max 100}]]
   [:bio {:optional true} [:string {:max 500}]]
   [:preferences {:optional true}
    [:map
     [:theme {:optional true} [:enum "light" "dark"]]
     [:language {:optional true} [:enum "en" "fi" "sv"]]]]])

;; Response schemas
(def UserResponse
  [:map
   [:id uuid?]
   [:email string?]
   [:name string?]
   [:created-at inst?]])

(def ApiError
  [:map
   [:error string?]
   [:message string?]
   [:details {:optional true} any?]])

(def PaginatedResponse
  [:map
   [:data [:vector any?]]
   [:total nat-int?]
   [:page nat-int?]
   [:per-page nat-int?]])

;; Specific paginated response
(defn paginated [item-schema]
  [:map
   [:data [:vector item-schema]]
   [:total nat-int?]
   [:page nat-int?]
   [:per-page nat-int?]])

(def PaginatedUsers (paginated UserResponse))
```

### 3. Configuration Schemas

```clojure
(ns myapp.schemas.config
  (:require [malli.core :as m]))

(def DatabaseConfig
  [:map
   [:host string?]
   [:port [:int {:min 1 :max 65535}]]
   [:database string?]
   [:username string?]
   [:password string?]
   [:pool-size {:optional true} [:int {:min 1 :max 100}]]
   [:timeout-ms {:optional true} pos-int?]])

(def ServerConfig
  [:map
   [:port [:int {:min 1 :max 65535}]]
   [:host string?]
   [:env [:enum "dev" "staging" "production"]]
   [:log-level [:enum "trace" "debug" "info" "warn" "error"]]])

(def AppConfig
  [:map
   [:server ServerConfig]
   [:database DatabaseConfig]
   [:feature-flags {:optional true}
    [:map-of keyword? boolean?]]])
```

### 4. Sum Types and Polymorphic Data

```clojure
(ns myapp.schemas.events
  (:require [malli.core :as m]))

;; Discriminated union
(def Event
  [:multi {:dispatch :type}
   [:user/created
    [:map
     [:type [:= :user/created]]
     [:user-id uuid?]
     [:email string?]
     [:timestamp inst?]]]
   [:user/updated
    [:map
     [:type [:= :user/updated]]
     [:user-id uuid?]
     [:changes [:map-of keyword? any?]]
     [:timestamp inst?]]]
   [:user/deleted
    [:map
     [:type [:= :user/deleted]]
     [:user-id uuid?]
     [:timestamp inst?]]]])

;; Usage
(m/validate Event
  {:type :user/created
   :user-id #uuid "123e4567-e89b-12d3-a456-426614174000"
   :email "user@example.com"
   :timestamp #inst "2025-01-01T00:00:00Z"})
;; => true
```

## Schema Registry

### Setting Up a Central Registry

```clojure
(ns myapp.schemas.registry
  (:require [malli.core :as m]
            [malli.registry :as mr]))

;; Define registry
(def registry
  (merge
   (m/default-schemas)
   {:user/id uuid?
    :user/email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]
    :user/name [:string {:min 1 :max 100}]
    :user/age [:int {:min 0 :max 150}]

    ;; Domain entities
    ::user
    [:map
     [:id :user/id]
     [:email :user/email]
     [:name :user/name]
     [:age {:optional true} :user/age]]

    ::address
    [:map
     [:street string?]
     [:city string?]
     [:postal-code [:string {:min 5 :max 10}]]
     [:country [:enum "FI" "SE" "NO" "DK"]]]

    ::order
    [:map
     [:id uuid?]
     [:user-id :user/id]
     [:items [:vector ::order-item]]
     [:total [:double {:min 0}]]
     [:status [:enum :pending :processing :completed :cancelled]]]

    ::order-item
    [:map
     [:product-id uuid?]
     [:quantity pos-int?]
     [:price [:double {:min 0}]]]}))

;; Set as default registry
(mr/set-default-registry! registry)

;; Now you can use registry references
(m/validate ::user
  {:id #uuid "123e4567-e89b-12d3-a456-426614174000"
   :email "test@example.com"
   :name "Test User"})
;; => true
```

### Using the Registry

```clojure
(ns myapp.core
  (:require [malli.core :as m]
            [myapp.schemas.registry :as registry]))

;; Registry is already set as default, just use the refs
(defn create-user [user-data]
  (when (m/validate ::registry/user user-data)
    (save-to-db! user-data)))

;; Or pass registry explicitly
(defn validate-order [order]
  (m/validate ::registry/order order
    {:registry registry/registry}))
```

## Validation and Error Handling

### Basic Validation

```clojure
(require '[malli.core :as m]
         '[malli.error :as me])

(def UserSchema
  [:map
   [:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]
   [:age [:int {:min 0 :max 150}]]])

;; Simple validation
(m/validate UserSchema {:email "test@example.com" :age 30})
;; => true

(m/validate UserSchema {:email "invalid" :age 30})
;; => false

;; Get detailed errors
(-> UserSchema
    (m/explain {:email "invalid" :age 200})
    (me/humanize))
;; => {:email ["should match regex"]
;;     :age ["should be at most 150"]}
```

### Custom Error Messages

```clojure
(def UserSchemaWithMessages
  [:map
   [:email {:error/message "Must be a valid email address"}
    [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]
   [:age {:error/message "Age must be between 0 and 150"}
    [:int {:min 0 :max 150}]]
   [:name {:error/message "Name is required and must be 1-100 characters"}
    [:string {:min 1 :max 100}]]])

(-> UserSchemaWithMessages
    (m/explain {:email "invalid" :age 200 :name ""})
    (me/humanize))
;; => {:email ["Must be a valid email address"]
;;     :age ["Age must be between 0 and 150"]
;;     :name ["Name is required and must be 1-100 characters"]}
```

### Function for Validation with Error Details

```clojure
(defn validate-with-errors [schema data]
  (if (m/validate schema data)
    {:valid? true :data data}
    {:valid? false
     :errors (-> schema
                 (m/explain data)
                 (me/humanize))}))

;; Usage
(validate-with-errors UserSchema {:email "test@example.com" :age 30})
;; => {:valid? true, :data {:email "test@example.com", :age 30}}

(validate-with-errors UserSchema {:email "invalid" :age 200})
;; => {:valid? false, :errors {:email [...], :age [...]}}
```

## Data Transformation and Coercion

Malli provides powerful transformation capabilities for converting data between different representations (JSON, strings, EDN, etc.) while maintaining schema validity.

### Built-in Transformers

```clojure
(require '[malli.transform :as mt])

(def UserSchema
  [:map
   [:id [:string]]
   [:age [:int]]
   [:active [:boolean]]])

;; String transformer (useful for JSON/API input)
(m/decode UserSchema
  {:id "123" :age "30" :active "true"}
  (mt/string-transformer))
;; => {:id "123", :age 30, :active true}

;; JSON transformer
(m/decode UserSchema
  {:id "123" :age "30" :active true}
  (mt/json-transformer))
;; => {:id "123", :age 30, :active true}

;; Strip extra keys
(m/decode UserSchema
  {:id "123" :age 30 :active true :extra-key "ignored"}
  (mt/strip-extra-keys-transformer))
;; => {:id "123", :age 30, :active true}

;; Default value transformer
(m/decode [:map
           [:id :string]
           [:count {:default 0} :int]
           [:active {:default true} :boolean]]
  {:id "123"}
  (mt/default-value-transformer))
;; => {:id "123", :count 0, :active true}
```

### Custom Transformers

```clojure
(def trim-strings-transformer
  (mt/transformer
    {:name :trim-strings}
    {:decoders
     {:string {:compile (fn [_schema _]
                          (fn [x]
                            (if (string? x)
                              (clojure.string/trim x)
                              x)))}}}))

(m/decode [:string {:min 1}]
  "  hello  "
  trim-strings-transformer)
;; => "hello"

;; Combine transformers
(def api-transformer
  (mt/transformer
    (mt/string-transformer)
    (mt/strip-extra-keys-transformer)
    (mt/default-value-transformer)
    trim-strings-transformer))
```

**See `references/transformations.md` for comprehensive transformation patterns and examples.**

## Function Schemas and Instrumentation

Function schemas provide runtime validation of function inputs and outputs, with integration for static analysis tools like clj-kondo.

### Define Function Schemas

```clojure
(require '[malli.core :as m]
         '[malli.instrument :as mi])

;; Define function schema
(def create-user-schema
  [:=>
   [:cat [:map [:email string?] [:name string?]]]
   [:map [:id uuid?] [:email string?] [:name string?]]])

;; Associate with function
(defn create-user [user-data]
  (assoc user-data :id (random-uuid)))

(m/=> create-user create-user-schema)

;; Instrument for development (only validates when enabled)
(mi/instrument!)

;; Now function calls are validated
(create-user {:email "test@example.com" :name "Test"})
;; Works fine

(create-user {:email 123 :name "Test"})
;; Throws validation error
```

### Multiple Arity Functions

```clojure
(def update-user-schema
  [:function
   [:=> [:cat uuid? [:map-of keyword? any?]] ::user]
   [:=> [:cat uuid? [:map-of keyword? any?] [:map]] ::user]])

(defn update-user
  ([user-id changes]
   (update-user user-id changes {}))
  ([user-id changes opts]
   ;; implementation
   ))

(m/=> update-user update-user-schema)
```

### Shorthand Syntax

```clojure
;; Use :-> for simpler function schemas
[:-> :int :int]                      ; int → int
[:-> :string :int :string]           ; (string, int) → string
```

**See `references/function-schemas.md` for comprehensive function schema patterns and instrumentation strategies.**

## Generative Testing

Malli provides powerful generative testing capabilities built on test.check, automatically generating test data that conforms to your schemas.

### Basic Generation

```clojure
(require '[malli.generator :as mg])

;; Generate sample data
(mg/generate UserSchema)
;; => {:email "xk@d.v", :age 42}

(mg/sample UserSchema {:size 5})
;; => Generate 5 samples

(mg/generate [:vector UserSchema] {:size 3})
;; => [{:email "a@b.c", :age 10}
;;     {:email "x@y.z", :age 55}
;;     {:email "p@q.r", :age 23}]

;; Reproducible generation with seed
(mg/generate UserSchema {:seed 42})
;; Always generates the same value
```

### Property-Based Testing

```clojure
(require '[clojure.test.check.properties :as prop]
         '[clojure.test.check.clojure-test :refer [defspec]])

(def UserSchema
  [:map
   [:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]
   [:age [:int {:min 0 :max 150}]]])

;; Test that generated users always validate
(defspec user-validation-test 100
  (prop/for-all [user (mg/generator UserSchema)]
    (m/validate UserSchema user)))

;; Test business logic with generated data
(defspec user-age-calculation-test 100
  (prop/for-all [user (mg/generator UserSchema)]
    (let [birth-year (- 2025 (:age user))]
      (>= birth-year 1875))))

;; Roundtrip testing (encode/decode)
(defspec string-transformer-roundtrip 100
  (prop/for-all [data (mg/generator UserSchema)]
    (let [encoded (m/encode UserSchema data (mt/string-transformer))
          decoded (m/decode UserSchema encoded (mt/string-transformer))]
      (= data decoded))))
```

### Custom Generators

```clojure
(require '[malli.generator :as mg]
         '[clojure.test.check.generators :as gen])

(def UserSchemaWithCustomGen
  [:map
   [:id [:string {:gen/gen (gen/fmap str (gen/uuid))}]]
   [:email [:string {:gen/gen (gen/fmap
                                #(str % "@example.com")
                                (gen/such-that
                                  not-empty
                                  (gen/string-alphanumeric)))}]]
   [:age [:int {:min 18 :max 65}]]])

(mg/generate UserSchemaWithCustomGen)
;; => {:id "123e4567-e89b-12d3-a456-426614174000"
;;     :email "abc123@example.com"
;;     :age 42}
```

**See `references/generation.md` for comprehensive generative testing patterns and examples.**

## Integration with Guardrails

### Shared Schema Definitions

```clojure
(ns myapp.schemas.user
  (:require [malli.core :as m]))

;; Define once
(def User
  [:map
   [:id uuid?]
   [:email string?]
   [:name string?]])

;; Use in Guardrails
(ns myapp.api.users
  (:require [guardrails.core :refer [>defn]]
            [myapp.schemas.user :as user-schemas]))

(>defn create-user
  [user-data]
  [user-schemas/User => user-schemas/User]
  ;; implementation
  (assoc user-data :id (random-uuid)))
```

### Schema-First Development Workflow

1. Define Malli schemas in `schemas/` namespace
2. Use schemas for API validation
3. Reference same schemas in Guardrails `>defn`
4. Generate tests from schemas
5. Document schemas in vault

```clojure
;; 1. Define schema
(ns myapp.schemas.order
  (:require [malli.core :as m]))

(def Order
  [:map
   [:id uuid?]
   [:items [:vector ::order-item]]
   [:total [:double {:min 0}]]])

;; 2. API validation
(ns myapp.api.orders
  (:require [malli.core :as m]
            [myapp.schemas.order :as order-schemas]))

(defn create-order-handler [request]
  (let [order-data (get-in request [:body :order])]
    (if (m/validate order-schemas/Order order-data)
      {:status 200 :body (create-order order-data)}
      {:status 400 :body {:error "Invalid order data"}})))

;; 3. Guardrails contracts
(ns myapp.domain.orders
  (:require [guardrails.core :refer [>defn]]
            [myapp.schemas.order :as order-schemas]))

(>defn process-order
  [order]
  [order-schemas/Order => order-schemas/Order]
  ;; processing logic
  )

;; 4. Generative tests
(ns myapp.domain.orders-test
  (:require [clojure.test :refer :all]
            [malli.generator :as mg]
            [myapp.schemas.order :as order-schemas]
            [myapp.domain.orders :as orders]))

(deftest process-order-test
  (testing "Process order maintains schema validity"
    (doseq [order (mg/sample order-schemas/Order 100)]
      (is (m/validate order-schemas/Order
                      (orders/process-order order))))))
```

## Documentation Conventions

### Document Schemas in Vault

Create `vault/specifications/schemas.md`:

```markdown
# Data Schemas

This document describes the data schemas used throughout the application.

## User Schemas

### User Entity

**Location:** `myapp.schemas.user/User`

**Purpose:** Core user entity representing authenticated users.

**Schema:**
```clojure
[:map
 [:id uuid?]
 [:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]
 [:name [:string {:min 1 :max 100}]]
 [:age {:optional true} [:int {:min 0 :max 150}]]
 [:created-at inst?]]
```

**Example:**
```clojure
{:id #uuid "123e4567-e89b-12d3-a456-426614174000"
 :email "user@example.com"
 :name "John Doe"
 :age 30
 :created-at #inst "2025-01-01T00:00:00Z"}
```

**Validation Rules:**
- Email must be valid format
- Name must be 1-100 characters
- Age is optional but must be 0-150 if provided

**Related Schemas:** `UserProfile`, `UserResponse`
```

### Schema Organization

```
src/
└── myapp/
    └── schemas/
        ├── registry.clj      # Central registry
        ├── user.clj          # User-related schemas
        ├── order.clj         # Order-related schemas
        ├── api.clj           # API request/response schemas
        └── config.clj        # Configuration schemas
```

## Best Practices

### 1. Use Namespaced Keywords

```clojure
;; Good - namespaced keywords prevent collisions
(def User
  [:map
   [:user/id uuid?]
   [:user/email string?]
   [:user/name string?]])

;; Less ideal - generic keywords
(def User
  [:map
   [:id uuid?]
   [:email string?]
   [:name string?]])
```

### 2. Extract Common Patterns

```clojure
;; Define reusable patterns
(def email-regex #"^[^\s@]+@[^\s@]+\.[^\s@]+$")
(def Email [:re {:error/message "Invalid email format"} email-regex])

(def NonEmptyString [:string {:min 1 :error/message "Cannot be empty"}])

(def PositiveDouble [:double {:min 0 :error/message "Must be positive"}])

;; Use in schemas
(def User
  [:map
   [:email Email]
   [:name NonEmptyString]
   [:balance PositiveDouble]])
```

### 3. Provide Good Error Messages

```clojure
(def Order
  [:map
   [:items {:error/message "Order must contain at least one item"}
    [:vector {:min 1} ::order-item]]
   [:total {:error/message "Total must be a positive amount"}
    [:double {:min 0.01}]]
   [:customer-id {:error/message "Customer ID is required"}
    uuid?]])
```

### 4. Version Your Schemas

```clojure
(ns myapp.schemas.user.v1)
(def User [:map [:id uuid?] [:name string?]])

(ns myapp.schemas.user.v2)
(def User [:map [:id uuid?] [:name string?] [:email string?]])

;; Use aliases for current version
(ns myapp.schemas.user
  (:require [myapp.schemas.user.v2 :as current]))

(def User current/User)
```

### 5. Test Your Schemas

```clojure
(ns myapp.schemas.user-test
  (:require [clojure.test :refer :all]
            [malli.core :as m]
            [myapp.schemas.user :as user-schemas]))

(deftest user-schema-test
  (testing "Valid user"
    (is (m/validate user-schemas/User
          {:id #uuid "123e4567-e89b-12d3-a456-426614174000"
           :email "test@example.com"
           :name "Test User"})))

  (testing "Invalid email"
    (is (not (m/validate user-schemas/User
               {:id #uuid "123e4567-e89b-12d3-a456-426614174000"
                :email "invalid"
                :name "Test User"}))))

  (testing "Missing required field"
    (is (not (m/validate user-schemas/User
               {:id #uuid "123e4567-e89b-12d3-a456-426614174000"
                :name "Test User"})))))
```

## Common Schema Templates

### Timestamped Entity

```clojure
(defn timestamped [entity-schema]
  (mu/merge entity-schema
    [:map
     [:created-at inst?]
     [:updated-at inst?]]))

(def User
  (timestamped
    [:map
     [:id uuid?]
     [:email string?]
     [:name string?]]))
```

### Paginated Collection

```clojure
(defn paginated [item-schema]
  [:map
   [:items [:vector item-schema]]
   [:page nat-int?]
   [:per-page pos-int?]
   [:total nat-int?]])
```

### API Response Wrapper

```clojure
(defn api-response [data-schema]
  [:map
   [:success boolean?]
   [:data data-schema]
   [:timestamp inst?]])
```

## Quality Requirements Integration

When defining schemas, link to quality requirements:

```clojure
(def UserPassword
  [:string
   {:min 8
    :max 128
    :error/message "Password must meet security requirements"
    :doc "See vault/requirements/security.md for password policy"}
   [:re #"^(?=.*[A-Z])(?=.*[a-z])(?=.*\d).*$"]])
```

## Key Advantages of Malli

1. **Schemas as Data**: First-class data structures (not macros) that can be manipulated, composed, and generated programmatically
2. **Multiple Syntaxes**: Both vector syntax (convenient) and map syntax (AST) for different use cases
3. **Comprehensive Type System**: Rich built-in types plus easy custom types
4. **Performance**: Optimized validators for production use
5. **Transformations**: Built-in support for coercion, encoding, decoding with composable transformers
6. **Generative Testing**: Seamless integration with test.check for property-based testing
7. **Function Schemas**: Runtime validation and static analysis integration
8. **Multiple Formats**: Generate JSON Schema, Swagger/OpenAPI, documentation from schemas
9. **Full Stack**: Works in Clojure, ClojureScript, Babashka, and GraalVM
10. **Active Maintenance**: Well-maintained by Metosin with commercial support available

## Workflow Tips

1. **Start with Registry**: Define a central registry for your domain schemas
2. **Schema-First Development**: Define schemas before implementation
3. **Instrument in Development**: Enable function instrumentation only during development
4. **Test with Generators**: Use property-based testing to validate business logic
5. **Document in Vault**: Keep schema documentation in vault for team reference
6. **Version Schemas**: Use namespaced versions when schemas evolve
7. **Validate Early**: Use transformers at API boundaries to validate and coerce input
8. **Test Transformations**: Use roundtrip tests to ensure encode/decode are inverses

## References

**Internal Reference Documentation:**
- `references/schema-types.md` - Complete schema type reference
- `references/transformations.md` - Transformation and coercion guide
- `references/generation.md` - Generative testing guide
- `references/function-schemas.md` - Function schema reference

**External Resources:**
- Malli documentation: https://github.com/metosin/malli
- Malli examples: https://github.com/metosin/malli/tree/master/docs
- API documentation: https://cljdoc.org/d/metosin/malli/
- Integration with Guardrails: See `guardrails-contracts.md`
