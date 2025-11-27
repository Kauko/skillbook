# Malli Generative Testing and Generators Reference

Complete guide to generating test data and property-based testing with Malli.

## Overview

Malli provides powerful generative testing capabilities built on top of `test.check`, allowing you to automatically generate test data that conforms to your schemas.

## Basic Generation

### Generate Function

Generate a single random value matching a schema:

```clojure
(require '[malli.generator :as mg])

(mg/generate :int)
;; => 42

(mg/generate :string)
;; => "abc"

(mg/generate [:map [:id :int] [:name :string]])
;; => {:id 10, :name "xyz"}

(mg/generate [:vector :int])
;; => [1 5 -3 8 2]
```

### Sample Function

Generate multiple values:

```clojure
(mg/sample :int)
;; => (0 1 -1 2 3 -2 4 5 -3 6)

(mg/sample :int {:size 5})
;; => (0 1 -1 2 3)

(mg/sample [:map [:id :int] [:active :boolean]] {:size 3})
;; => ({:id 0, :active true}
;;     {:id -1, :active false}
;;     {:id 1, :active true})
```

### Generator Function

Create a reusable generator:

```clojure
(def user-generator
  (mg/generator
    [:map
     [:id uuid?]
     [:email :string]
     [:age [:int {:min 18 :max 100}]]]))

(mg/generate user-generator)
;; => {:id #uuid "...", :email "abc", :age 42}

; Use with test.check
(require '[clojure.test.check.generators :as gen])

(gen/generate user-generator)
;; => {:id #uuid "...", :email "xyz", :age 55}
```

## Generator Options

### Size Control

Control the size of generated collections:

```clojure
(mg/generate [:vector :int] {:size 5})
;; => [1 2 3 4 5]

(mg/generate [:string] {:size 10})
;; => "abcdefghij"

(mg/sample [:set :keyword] {:size 3})
;; => (#{:a} #{:b :c} #{:d :e :f})
```

### Seed for Reproducibility

Use seeds for reproducible generation:

```clojure
(mg/generate [:vector :int] {:seed 42})
;; Always generates the same value with seed 42

(mg/sample :string {:seed 123, :size 5})
;; Reproducible sample generation
```

## Schema-Specific Generation

### Primitive Types

```clojure
(mg/generate :int)          ;; => 42
(mg/generate :string)       ;; => "abc"
(mg/generate :boolean)      ;; => true
(mg/generate :keyword)      ;; => :keyword
(mg/generate uuid?)         ;; => #uuid "..."
(mg/generate inst?)         ;; => #inst "..."
```

### Numeric Constraints

```clojure
(mg/generate [:int {:min 0 :max 100}])
;; => 55

(mg/generate [:double {:min 0.0 :max 1.0}])
;; => 0.7234

(mg/sample pos-int? {:size 5})
;; => (1 2 3 4 5)

(mg/generate nat-int?)
;; => 42
```

### String Constraints

```clojure
(mg/generate [:string {:min 5 :max 10}])
;; => "abcdefgh"

(mg/generate [:re #"^[a-z]{3}$"])
;; => "abc"

(mg/generate [:re #"^[0-9]{3}-[0-9]{2}-[0-9]{4}$"])
;; => "123-45-6789"
```

### Collections

```clojure
(mg/generate [:vector :int])
;; => [1 5 -3 8 2]

(mg/generate [:vector {:min 2 :max 5} :int])
;; => [1 5 -3]

(mg/generate [:set :keyword])
;; => #{:a :b :c}

(mg/generate [:map [:id :int] [:name :string]])
;; => {:id 42, :name "abc"}

(mg/generate [:tuple :string :int :boolean])
;; => ["abc" 42 true]
```

### Enums

```clojure
(mg/generate [:enum "red" "green" "blue"])
;; => "green"

(mg/generate [:enum :pending :processing :complete])
;; => :processing

(mg/sample [:enum 1 2 3] {:size 10})
;; => (1 2 3 1 2 3 1 2 3 1)
```

### Optional and Maybe

```clojure
(mg/sample [:maybe :string] {:size 10})
;; => (nil "a" nil "bc" "def" nil "g" nil "hij" "klm")

(mg/generate [:map
              [:id :int]
              [:email {:optional true} :string]])
;; => {:id 42} or {:id 42, :email "test@example.com"}
```

## Custom Generators

### Using :gen/gen Property

Attach custom generators to schemas:

```clojure
(require '[clojure.test.check.generators :as gen])

; Generate specific email format
(mg/generate
  [:string {:gen/gen (gen/fmap
                       #(str % "@example.com")
                       (gen/such-that not-empty gen/string-alphanumeric))}])
;; => "abc123@example.com"

; Generate UUID strings
(mg/generate
  [:string {:gen/gen (gen/fmap str gen/uuid)}])
;; => "123e4567-e89b-12d3-a456-426614174000"

; Generate realistic ages
(mg/generate
  [:int {:gen/gen (gen/choose 18 80)}])
;; => 42
```

### Custom Generator Functions

```clojure
; Generate realistic user data
(def realistic-user-generator
  [:map
   [:id {:gen/gen gen/uuid}]
   [:email {:gen/gen (gen/fmap
                       (fn [[name domain]]
                         (str name "@" domain ".com"))
                       (gen/tuple
                         (gen/such-that not-empty gen/string-alphanumeric)
                         (gen/elements ["gmail" "yahoo" "hotmail"])))}]
   [:age {:gen/gen (gen/choose 18 80)}]
   [:status {:gen/gen (gen/elements [:active :inactive :pending])}]])

(mg/generate realistic-user-generator)
;; => {:id #uuid "...",
;;     :email "abc123@gmail.com",
;;     :age 42,
;;     :status :active}
```

### Generator with Schema Properties

```clojure
(def EmailSchema
  [:string
   {:gen/gen (gen/fmap
               #(str % "@example.com")
               (gen/such-that not-empty gen/string-alphanumeric))
    :gen/min 5
    :gen/max 20}])

(mg/sample EmailSchema {:size 3})
;; => ("a@example.com" "bc@example.com" "def@example.com")
```

## Property-Based Testing

### Basic Property Test

```clojure
(require '[clojure.test.check.properties :as prop]
         '[clojure.test.check.clojure-test :refer [defspec]])

(def UserSchema
  [:map
   [:id uuid?]
   [:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]
   [:age [:int {:min 0 :max 150}]]])

; Test that generated users always validate
(defspec generated-users-are-valid 100
  (prop/for-all [user (mg/generator UserSchema)]
    (m/validate UserSchema user)))
```

### Testing Business Logic

```clojure
(defn calculate-discount [user]
  (cond
    (< (:age user) 18) 0.1
    (>= (:age user) 65) 0.15
    :else 0.0))

(defspec discount-calculation-test 100
  (prop/for-all [user (mg/generator [:map [:age [:int {:min 0 :max 150}]]])]
    (let [discount (calculate-discount user)]
      (and (>= discount 0.0)
           (<= discount 1.0)))))

; Test specific properties
(defspec senior-discount-test 100
  (prop/for-all [age (gen/choose 65 150)]
    (= 0.15 (calculate-discount {:age age}))))

(defspec child-discount-test 100
  (prop/for-all [age (gen/choose 0 17)]
    (= 0.1 (calculate-discount {:age age}))))
```

### Roundtrip Testing

Test that encode/decode operations are inverses:

```clojure
(require '[malli.transform :as mt])

(def Schema
  [:map
   [:id :string]
   [:age :int]
   [:active :boolean]])

(defspec string-transformer-roundtrip 100
  (prop/for-all [data (mg/generator Schema)]
    (let [encoded (m/encode Schema data (mt/string-transformer))
          decoded (m/decode Schema encoded (mt/string-transformer))]
      (= data decoded))))
```

### Testing Schema Transformations

```clojure
(def InputSchema
  [:map
   [:email [:string]]
   [:age [:string]]])

(def OutputSchema
  [:map
   [:email [:string]]
   [:age [:int]]])

(defn transform-user [input]
  (m/decode OutputSchema input (mt/string-transformer)))

(defspec transformation-preserves-structure 100
  (prop/for-all [input (mg/generator InputSchema)]
    (let [output (transform-user input)]
      (and (m/validate OutputSchema output)
           (= (:email input) (:email output))))))
```

## Advanced Generation Techniques

### Recursive Schemas

```clojure
(def TreeSchema
  [:map
   [:value :int]
   [:children {:optional true} [:vector [:ref ::TreeSchema]]]])

(mg/generate TreeSchema {:seed 42, :size 5})
;; => {:value 0, :children [{:value 1} {:value 2, :children [{:value 3}]}]}
```

### Multi Schemas (Discriminated Unions)

```clojure
(def EventSchema
  [:multi {:dispatch :type}
   [:created [:map
              [:type [:= :created]]
              [:user-id uuid?]
              [:timestamp inst?]]]
   [:updated [:map
              [:type [:= :updated]]
              [:user-id uuid?]
              [:changes [:map-of :keyword any?]]]]
   [:deleted [:map
              [:type [:= :deleted]]
              [:user-id uuid?]]]])

(mg/sample EventSchema {:size 5})
;; => ({:type :created, :user-id #uuid "...", :timestamp #inst "..."}
;;     {:type :updated, :user-id #uuid "...", :changes {:foo "bar"}}
;;     {:type :deleted, :user-id #uuid "..."}
;;     ...)
```

### Constrained Generation

```clojure
; Generate pairs where first < second
(def ordered-pair-gen
  (gen/fmap
    (fn [[a b]] (if (< a b) [a b] [b a]))
    (gen/tuple gen/int gen/int)))

(def OrderedPairSchema
  [:tuple
   {:gen/gen ordered-pair-gen}
   :int :int])

(mg/sample OrderedPairSchema {:size 5})
;; => ([0 1] [-1 0] [1 2] [-2 3] [0 5])
```

### Dependent Values

```clojure
; Generate user where email domain matches country
(def user-with-country-gen
  (gen/let [country (gen/elements ["US" "UK" "DE"])
            name (gen/such-that not-empty gen/string-alphanumeric)
            domain (case country
                     "US" "com"
                     "UK" "co.uk"
                     "DE" "de")]
    {:country country
     :name name
     :email (str name "@example." domain)}))

(def UserWithCountrySchema
  [:map {:gen/gen user-with-country-gen}
   [:country :string]
   [:name :string]
   [:email :string]])

(mg/generate UserWithCountrySchema)
;; => {:country "UK", :name "abc", :email "abc@example.co.uk"}
```

## Generator Configuration

### Global Generator Options

```clojure
; Set default size
(mg/generate schema {:size 100})

; Set seed for reproducibility
(mg/generate schema {:seed 42})

; Combine options
(mg/generate schema {:seed 42, :size 20})
```

### Schema-Level Generator Options

```clojure
(def Schema
  [:map
   [:tags [:vector {:gen/min 1 :gen/max 5} :keyword]]
   [:count [:int {:gen/min 0 :gen/max 100}]]
   [:name [:string {:gen/min 3 :gen/max 20}]]])

(mg/generate Schema)
;; Respects all gen/* constraints
```

## Testing Patterns

### Schema Validation Testing

```clojure
(deftest schema-validation-test
  (testing "Generated data always validates"
    (is (every?
          #(m/validate UserSchema %)
          (mg/sample UserSchema {:size 100})))))
```

### Regression Testing

```clojure
; Use fixed seeds for regression tests
(deftest specific-case-test
  (let [user (mg/generate UserSchema {:seed 12345})]
    (is (valid-user? user))))
```

### Edge Case Generation

```clojure
; Generate edge cases explicitly
(def edge-case-generators
  [(constantly {:age 0})
   (constantly {:age 150})
   (constantly {:age 18})
   (constantly {:age 65})])

(deftest edge-case-test
  (doseq [gen edge-case-generators]
    (let [user (gen)]
      (is (valid-user? user)))))
```

### Shrinking Failed Cases

test.check automatically shrinks failing cases to minimal examples:

```clojure
(defspec user-age-valid 100
  (prop/for-all [user (mg/generator UserSchema)]
    (<= 0 (:age user) 150)))

; If this fails, test.check will shrink to smallest failing case
```

## Best Practices

### 1. Use Realistic Generators

```clojure
; Good: Realistic email generator
[:string {:gen/gen (gen/fmap
                     #(str % "@example.com")
                     gen/string-alphanumeric)}]

; Less ideal: Generic string
:string
```

### 2. Test Invariants

```clojure
(defspec user-creation-preserves-email 100
  (prop/for-all [email (gen/such-that not-empty gen/string-alphanumeric)]
    (let [user (create-user {:email (str email "@example.com")})]
      (clojure.string/includes? (:email user) email))))
```

### 3. Generate Valid and Invalid Data

```clojure
; Test that validation rejects invalid data
(defspec invalid-age-rejected 100
  (prop/for-all [age (gen/choose -100 -1)]
    (not (m/validate [:int {:min 0 :max 150}] age))))
```

### 4. Use Seeds for Debugging

```clojure
; When a property test fails, note the seed
(defspec my-test 100
  (prop/for-all [x (mg/generator schema {:seed 42})]
    (my-property x)))
```

### 5. Combine with Unit Tests

```clojure
; Use generators to create test fixtures
(deftest user-creation-test
  (let [user-data (mg/generate UserInputSchema)]
    (is (= :created (:status (create-user user-data))))))
```

## Common Patterns

### Generating Test Fixtures

```clojure
(defn generate-test-users [n]
  (mg/sample UserSchema {:size n}))

(deftest batch-processing-test
  (let [users (generate-test-users 50)]
    (is (= 50 (count (process-users users))))))
```

### Generating Mock Data for Development

```clojure
(defn seed-database []
  (let [users (mg/sample UserSchema {:size 100 :seed 42})]
    (doseq [user users]
      (db/insert-user user))))
```

### Testing API Contracts

```clojure
(defspec api-response-matches-schema 100
  (prop/for-all [request (mg/generator ApiRequestSchema)]
    (let [response (api-handler request)]
      (m/validate ApiResponseSchema response))))
```

## Integration with test.check

### Using test.check Generators Directly

```clojure
(require '[clojure.test.check.generators :as gen])

; Combine test.check and Malli generators
(def custom-generator
  (gen/let [id gen/uuid
            name (gen/such-that not-empty gen/string-alphanumeric)
            tags (gen/vector (gen/elements [:tag1 :tag2 :tag3]) 1 5)]
    {:id id
     :name name
     :tags tags}))

(gen/generate custom-generator)
```

### Complex Generators

```clojure
(def user-generator
  (gen/let [id gen/uuid
            email (gen/fmap #(str % "@example.com")
                            (gen/such-that not-empty gen/string-alphanumeric))
            age (gen/choose 18 80)
            premium? gen/boolean
            credits (if premium?
                      (gen/choose 100 1000)
                      (gen/return 0))]
    {:id id
     :email email
     :age age
     :premium? premium?
     :credits credits}))

(gen/sample user-generator 5)
```
