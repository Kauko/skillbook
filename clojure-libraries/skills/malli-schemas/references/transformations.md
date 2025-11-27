# Malli Transformations Reference

Complete guide to data transformation, coercion, encoding, and decoding in Malli.

## Overview

Malli provides powerful transformation capabilities for converting data between different representations (JSON, strings, EDN, etc.) while maintaining schema validity.

## Core Transformation Functions

### Decode

Transform from external format to Clojure EDN:

```clojure
(require '[malli.core :as m]
         '[malli.transform :as mt])

(def Schema
  [:map
   [:id :string]
   [:age :int]
   [:active :boolean]])

; Decode string values to proper types
(m/decode Schema
  {:id "123" :age "30" :active "true"}
  (mt/string-transformer))
;; => {:id "123", :age 30, :active true}
```

### Encode

Transform from Clojure EDN to external format:

```clojure
(m/encode Schema
  {:id "123" :age 30 :active true}
  (mt/string-transformer))
;; => {:id "123", :age "30", :active "true"}
```

### Decoder and Encoder

Create optimized transformation functions:

```clojure
(def string-decoder (m/decoder Schema (mt/string-transformer)))
(def string-encoder (m/encoder Schema (mt/string-transformer)))

(string-decoder {:id "123" :age "30" :active "true"})
;; => {:id "123", :age 30, :active true}

(string-encoder {:id "123" :age 30 :active true})
;; => {:id "123", :age "30", :active "true"}
```

## Built-in Transformers

### String Transformer

Converts string representations to typed values:

```clojure
(mt/string-transformer)

; Examples:
(m/decode :int "42" (mt/string-transformer))
;; => 42

(m/decode :boolean "true" (mt/string-transformer))
;; => true

(m/decode :keyword "status" (mt/string-transformer))
;; => :status

(m/decode [:vector :int] ["1" "2" "3"] (mt/string-transformer))
;; => [1 2 3]
```

### JSON Transformer

Converts between JSON-compatible and Clojure representations:

```clojure
(mt/json-transformer)

(m/decode [:map [:created-at inst?]]
  {:created-at "2025-01-01T00:00:00Z"}
  (mt/json-transformer))
;; => {:created-at #inst "2025-01-01T00:00:00.000-00:00"}

(m/encode [:map [:created-at inst?]]
  {:created-at #inst "2025-01-01"}
  (mt/json-transformer))
;; => {:created-at "2025-01-01T00:00:00.000-00:00"}
```

### Strip Extra Keys Transformer

Removes keys not defined in schema:

```clojure
(mt/strip-extra-keys-transformer)

(m/decode [:map [:id :string] [:name :string]]
  {:id "123" :name "Test" :extra "data" :more "stuff"}
  (mt/strip-extra-keys-transformer))
;; => {:id "123", :name "Test"}
```

### Default Value Transformer

Fills in default values for missing keys:

```clojure
(mt/default-value-transformer)

(m/decode [:map
           [:id :string]
           [:count {:default 0} :int]
           [:active {:default true} :boolean]]
  {:id "123"}
  (mt/default-value-transformer))
;; => {:id "123", :count 0, :active true}
```

## Combining Transformers

Chain multiple transformers for complex transformations:

```clojure
; Combine transformers
(def api-transformer
  (mt/transformer
    (mt/string-transformer)
    (mt/strip-extra-keys-transformer)
    (mt/default-value-transformer)))

(m/decode [:map
           [:id :string]
           [:age :int]
           [:active {:default true} :boolean]]
  {:id "123" :age "30" :extra "ignored"}
  api-transformer)
;; => {:id "123", :age 30, :active true}
```

## Custom Transformers

### Basic Custom Transformer

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

(m/decode :string "  hello  " trim-strings-transformer)
;; => "hello"
```

### Transformer with Multiple Types

```clojure
(def normalize-transformer
  (mt/transformer
    {:name :normalize}
    {:decoders
     {:string {:compile (fn [_schema _]
                          (fn [x]
                            (when x
                              (-> x
                                  clojure.string/trim
                                  clojure.string/lower-case))))}
      :keyword {:compile (fn [_schema _]
                           (fn [x]
                             (when x
                               (-> x name clojure.string/lower-case keyword))))}}}))

(m/decode [:map
           [:email :string]
           [:status :keyword]]
  {:email "  TEST@EXAMPLE.COM  " :status :ACTIVE}
  normalize-transformer)
;; => {:email "test@example.com", :status :active}
```

### Schema-Specific Transformer

```clojure
(def phone-transformer
  (mt/transformer
    {:name :phone}
    {:decoders
     {:string {:compile (fn [schema _]
                          (when (= "phone" (:type (m/properties schema)))
                            (fn [x]
                              (clojure.string/replace x #"[^0-9]" ""))))}}}))

(m/decode [:string {:type "phone"}]
  "(555) 123-4567"
  phone-transformer)
;; => "5551234567"
```

## Transformation Stages

### Decode Pipeline

1. **Pre-decode**: Transform before validation
2. **Validation**: Check schema compliance
3. **Post-decode**: Transform after validation

```clojure
(def Schema
  [:map
   [:email [:string {:decode/string clojure.string/lower-case}]]
   [:age :int]])

(m/decode Schema
  {:email "TEST@EXAMPLE.COM" :age "30"}
  (mt/string-transformer))
;; => {:email "test@example.com", :age 30}
```

### Schema-Level Transformations

Attach transformations directly to schemas:

```clojure
(def UserSchema
  [:map
   [:email [:string {:decode/string clojure.string/lower-case
                     :decode/json clojure.string/lower-case}]]
   [:name [:string {:decode/string clojure.string/trim}]]
   [:age [:int {:decode/string #(Integer/parseInt %)}]]])

(m/decode UserSchema
  {:email "  TEST@EXAMPLE.COM  "
   :name "  John Doe  "
   :age "30"}
  (mt/string-transformer))
;; => {:email "test@example.com", :name "John Doe", :age 30}
```

## Common Transformation Patterns

### API Request Transformation

```clojure
(def api-request-transformer
  (mt/transformer
    (mt/string-transformer)
    (mt/strip-extra-keys-transformer)
    (mt/default-value-transformer)))

(def CreateUserRequest
  [:map
   [:email [:string {:decode/string clojure.string/lower-case}]]
   [:name [:string {:min 1}]]
   [:age {:optional true :default 0} :int]
   [:newsletter {:default false} :boolean]])

(defn process-request [data]
  (m/decode CreateUserRequest data api-request-transformer))

(process-request
  {:email "TEST@EXAMPLE.COM"
   :name "  John  "
   :age "30"
   :extra-field "ignored"})
;; => {:email "test@example.com", :name "  John  ", :age 30, :newsletter false}
```

### Database Output Transformation

```clojure
(def db-transformer
  (mt/transformer
    {:name :db}
    {:decoders
     {:map {:compile (fn [_schema _]
                       (fn [x]
                         (reduce-kv
                           (fn [acc k v]
                             (assoc acc (keyword k) v))
                           {}
                           x)))}
      :keyword {:compile (fn [_schema _]
                           (fn [x]
                             (when x (keyword x))))}}}))

(m/decode [:map [:status :keyword] [:count :int]]
  {"status" "active" "count" 42}
  db-transformer)
;; => {:status :active, :count 42}
```

### Configuration Transformation

```clojure
(def config-transformer
  (mt/transformer
    (mt/string-transformer)
    (mt/default-value-transformer)))

(def ConfigSchema
  [:map
   [:host {:default "localhost"} :string]
   [:port {:default 3000} :int]
   [:debug {:default false} :boolean]
   [:workers {:default 4} :int]])

(m/decode ConfigSchema
  {:port "8080" :debug "true"}
  config-transformer)
;; => {:host "localhost", :port 8080, :debug true, :workers 4}
```

## Error Handling During Transformation

### Validate After Transformation

```clojure
(defn safe-decode [schema data transformer]
  (let [decoded (m/decode schema data transformer)]
    (if (m/validate schema decoded)
      {:success true :data decoded}
      {:success false
       :errors (-> schema
                   (m/explain decoded)
                   (me/humanize))})))

(safe-decode [:map [:age :int]]
  {:age "not-a-number"}
  (mt/string-transformer))
;; => {:success false, :errors {:age ["should be an integer"]}}
```

### Transform with Validation

```clojure
(require '[malli.error :as me])

(defn decode-and-validate [schema data transformer]
  (let [decoded (m/decode schema data transformer)]
    (when-let [explanation (m/explain schema decoded)]
      (throw (ex-info "Validation failed after transformation"
                      {:errors (me/humanize explanation)
                       :data decoded})))
    decoded))

(try
  (decode-and-validate [:map [:age [:int {:min 0 :max 150}]]]
    {:age "200"}
    (mt/string-transformer))
  (catch Exception e
    (ex-data e)))
;; => {:errors {:age ["should be at most 150"]}, :data {:age 200}}
```

## Type-Specific Transformations

### Date/Time Transformations

```clojure
(def datetime-transformer
  (mt/transformer
    {:name :datetime}
    {:decoders
     {:inst {:compile (fn [_schema _]
                        (fn [x]
                          (cond
                            (inst? x) x
                            (string? x) (java.time.Instant/parse x)
                            (number? x) (java.util.Date. x)
                            :else x)))}}}))

(m/decode :inst "2025-01-01T00:00:00Z" datetime-transformer)
;; => #inst "2025-01-01T00:00:00.000-00:00"
```

### UUID Transformations

```clojure
(def uuid-transformer
  (mt/transformer
    {:name :uuid}
    {:decoders
     {uuid? {:compile (fn [_schema _]
                        (fn [x]
                          (cond
                            (uuid? x) x
                            (string? x) (java.util.UUID/fromString x)
                            :else x)))}}}))

(m/decode uuid? "123e4567-e89b-12d3-a456-426614174000" uuid-transformer)
;; => #uuid "123e4567-e89b-12d3-a456-426614174000"
```

### Keyword/String Transformations

```clojure
(def keyword-transformer
  (mt/transformer
    {:name :keyword}
    {:decoders
     {:keyword {:compile (fn [_schema _]
                           (fn [x]
                             (cond
                               (keyword? x) x
                               (string? x) (keyword x)
                               :else x)))}}
     :encoders
     {:keyword {:compile (fn [_schema _]
                           (fn [x]
                             (when x (name x))))}}}))

(m/decode :keyword "status" keyword-transformer)
;; => :status

(m/encode :keyword :status keyword-transformer)
;; => "status"
```

## Advanced Transformation Techniques

### Conditional Transformations

```clojure
(def conditional-transformer
  (mt/transformer
    {:name :conditional}
    {:decoders
     {:string {:compile (fn [schema _]
                          (let [props (m/properties schema)]
                            (cond
                              (:uppercase props)
                              clojure.string/upper-case

                              (:lowercase props)
                              clojure.string/lower-case

                              (:trim props)
                              clojure.string/trim

                              :else identity)))}}}))

(m/decode [:string {:uppercase true}] "hello" conditional-transformer)
;; => "HELLO"

(m/decode [:string {:trim true}] "  hello  " conditional-transformer)
;; => "hello"
```

### Nested Transformations

```clojure
(def nested-transformer
  (mt/transformer
    (mt/string-transformer)
    (mt/strip-extra-keys-transformer)))

(m/decode [:map
           [:user [:map
                   [:id :string]
                   [:age :int]]]
           [:metadata [:map
                       [:created-at :string]
                       [:count :int]]]]
  {:user {:id "123" :age "30" :extra "field"}
   :metadata {:created-at "2025-01-01" :count "5"}}
  nested-transformer)
;; => {:user {:id "123", :age 30}
;;     :metadata {:created-at "2025-01-01", :count 5}}
```

## Best Practices

### 1. Create Reusable Transformers

```clojure
(ns myapp.transformers
  (:require [malli.transform :as mt]))

(def api-input-transformer
  "Standard transformer for API input"
  (mt/transformer
    (mt/string-transformer)
    (mt/strip-extra-keys-transformer)
    (mt/default-value-transformer)))

(def db-output-transformer
  "Transform database results to domain objects"
  (mt/transformer
    (mt/json-transformer)))
```

### 2. Validate After Transformation

```clojure
(defn decode! [schema data transformer]
  (let [decoded (m/decode schema data transformer)]
    (if (m/validate schema decoded)
      decoded
      (throw (ex-info "Invalid data after transformation"
                      {:errors (me/humanize (m/explain schema decoded))})))))
```

### 3. Use Schema Properties for Transformation Hints

```clojure
(def Schema
  [:map
   [:email [:string {:decode/string clojure.string/lower-case
                     :decode/json clojure.string/lower-case}]]
   [:phone [:string {:decode/string #(clojure.string/replace % #"[^0-9]" "")}]]])
```

### 4. Combine Transformers Thoughtfully

```clojure
; Good: Specific to general
(mt/transformer
  (mt/string-transformer)
  (mt/default-value-transformer)
  (mt/strip-extra-keys-transformer))

; Order matters for correctness
```

## Common Use Cases

### HTTP API Input

```clojure
(defn parse-api-input [schema json-data]
  (-> schema
      (m/decode json-data
        (mt/transformer
          (mt/json-transformer)
          (mt/string-transformer)
          (mt/strip-extra-keys-transformer)
          (mt/default-value-transformer)))))
```

### Form Data Processing

```clojure
(defn parse-form-data [schema form-params]
  (m/decode schema form-params
    (mt/transformer
      (mt/string-transformer)
      (mt/strip-extra-keys-transformer))))
```

### Configuration Loading

```clojure
(defn load-config [schema env-map]
  (m/decode schema env-map
    (mt/transformer
      (mt/string-transformer)
      (mt/default-value-transformer))))
```
