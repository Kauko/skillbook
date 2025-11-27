# Malli Schema Types Reference

Complete reference for all built-in Malli schema types and their usage.

## Primitive Types

### Basic Types

```clojure
:string          ; Any string
:int             ; Integer
:double          ; Double precision floating point
:boolean         ; Boolean value
:keyword         ; Keyword
:symbol          ; Symbol
:any             ; Accepts any value
:nil             ; Accepts only nil
```

### Predicate-Based Types

```clojure
int?             ; Integer predicate
string?          ; String predicate
boolean?         ; Boolean predicate
keyword?         ; Keyword predicate
symbol?          ; Symbol predicate
uuid?            ; UUID
inst?            ; Instance/Date
```

### Numeric Predicates

```clojure
pos-int?         ; Positive integer (> 0)
nat-int?         ; Natural number (>= 0)
neg-int?         ; Negative integer (< 0)
pos?             ; Positive number
neg?             ; Negative number
number?          ; Any number
double?          ; Double
float?           ; Float
```

## String Types

### Basic String Schemas

```clojure
:string                              ; Any string
[:string {:min 1}]                   ; Non-empty string
[:string {:min 1, :max 100}]         ; Length constrained
[:string {:min 5, :max 20}]          ; Range constraint
```

### Regular Expression Patterns

```clojure
[:re #"^[a-z]+$"]                    ; Lowercase letters only
[:re #"^[0-9]{5}$"]                  ; 5 digits
[:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]  ; Email pattern
[:re {:error/message "Invalid format"} #"^\d{3}-\d{2}-\d{4}$"]  ; SSN with error message
```

## Numeric Types

### Integer Schemas

```clojure
:int                                 ; Any integer
[:int {:min 0}]                      ; Non-negative
[:int {:min 0, :max 150}]            ; Range constraint
[:int {:min 1, :max 100}]            ; Positive range
```

### Double Schemas

```clojure
:double                              ; Any double
[:double {:min 0}]                   ; Non-negative
[:double {:min 0.0, :max 1.0}]       ; Percentage range
[:double {:min 0.01}]                ; Positive with precision
```

### Comparison Operators

```clojure
[:> 0]                               ; Greater than 0
[:>= 18]                             ; Greater than or equal to 18
[:< 100]                             ; Less than 100
[:<= 150]                            ; Less than or equal to 150
[:not= 0]                            ; Not equal to 0
```

## Collection Types

### Vector Schemas

```clojure
:vector                              ; Any vector
[:vector any?]                       ; Vector of anything
[:vector :int]                       ; Vector of integers
[:vector :string]                    ; Vector of strings
[:vector {:min 1} :int]              ; Non-empty vector of integers
[:vector {:min 1, :max 10} :string]  ; Size-constrained vector
```

### Set Schemas

```clojure
:set                                 ; Any set
[:set :keyword]                      ; Set of keywords
[:set :int]                          ; Set of integers
[:set {:min 1} :string]              ; Non-empty set
```

### Tuple Schemas

Fixed-length heterogeneous sequences:

```clojure
[:tuple :string :int]                ; [string, int]
[:tuple :double :double]             ; Coordinate pair
[:tuple :keyword :string :int]       ; Three-element tuple
[:tuple {:title "location"} :double :double]  ; With metadata
```

### Map Schemas

```clojure
; Basic map
[:map
 [:id :string]
 [:name :string]]

; With optional keys
[:map
 [:id :string]
 [:name :string]
 [:email {:optional true} :string]]

; With properties on map
[:map {:closed true}
 [:id :string]
 [:name :string]]

; Nested maps
[:map
 [:user [:map
   [:id :string]
   [:name :string]]]
 [:preferences [:map
   [:theme :keyword]
   [:notifications :boolean]]]]

; Map with qualified keys
[:map
 [:user/id :string]
 [:user/email :string]]

; Map with default handler
[:map {:registry {::default :string}}
 [:known-key :int]]
```

### Map-of Schemas

Homogeneous key-value pairs:

```clojure
[:map-of :keyword :string]           ; Keyword → String
[:map-of :string :int]               ; String → Integer
[:map-of :keyword any?]              ; Keyword → Any
[:map-of :string [:vector :int]]     ; String → Vector of Ints
```

### Sequential Collections

```clojure
:sequential                          ; Any sequential
[:sequential :int]                   ; Sequential of integers
:seqable                             ; Any seqable
[:seqable :string]                   ; Seqable of strings
```

## Logical Combinators

### And Schema

All conditions must be satisfied:

```clojure
[:and :int [:> 0] [:< 100]]          ; Integer between 0 and 100
[:and :string [:re #"^[A-Z]"]]       ; String starting with uppercase
[:and [:vector :int] [:min 1]]       ; Non-empty vector of integers
```

### Or Schema

At least one condition must be satisfied:

```clojure
[:or :int :string]                   ; Integer or string
[:or :nil :string]                   ; Nil or string (same as [:maybe :string])
[:or [:= 0] pos-int?]                ; Zero or positive integer
```

### Not Schema

Value must not match the schema:

```clojure
[:not :nil]                          ; Anything except nil
[:not [:= 0]]                        ; Anything except 0
```

## Specialized Types

### Enum

Enumerated values:

```clojure
[:enum 1 2 3]                        ; One of these integers
[:enum "red" "green" "blue"]         ; Color enum
[:enum :pending :processing :done]   ; Status enum
[:enum "FI" "SE" "NO" "DK"]          ; Country codes
```

### Equality

Exact value matching:

```clojure
[:= 42]                              ; Exactly 42
[:= "exact"]                         ; Exactly "exact"
[:= :keyword]                        ; Exactly :keyword
[:= nil]                             ; Exactly nil
```

### Maybe

Value or nil:

```clojure
[:maybe :string]                     ; String or nil
[:maybe :int]                        ; Integer or nil
[:maybe [:vector :keyword]]          ; Vector of keywords or nil
```

### Multi

Discriminated unions (polymorphic schemas):

```clojure
[:multi {:dispatch :type}
 [:user/created
  [:map
   [:type [:= :user/created]]
   [:user-id uuid?]
   [:email :string]]]
 [:user/updated
  [:map
   [:type [:= :user/updated]]
   [:user-id uuid?]
   [:changes [:map-of :keyword any?]]]]
 [:user/deleted
  [:map
   [:type [:= :user/deleted]]
   [:user-id uuid?]]]]
```

### Ref

Schema references (for recursive schemas):

```clojure
(def User
  [:map
   [:id :string]
   [:name :string]
   [:friends [:vector [:ref ::User]]]])

; With var references
(def Address [:map [:street :string]])
(def User [:map
  [:id :string]
  [:address #'Address]])
```

### Fn

Custom validation functions:

```clojure
[:fn (fn [x] (even? x))]             ; Even numbers
[:fn {:error/message "Must be palindrome"}
 (fn [s] (= s (clojure.string/reverse s)))]

; With multiple validators
[:and :string
 [:fn (fn [s] (> (count s) 5))]
 [:fn (fn [s] (re-matches #".*[A-Z].*" s))]]
```

### Schema

Reference to schema in registry:

```clojure
[:schema ::user]                     ; Reference to registered schema
[:schema {:registry {::id :string}} ::id]  ; With local registry
```

## Function Schemas

### Basic Function Types

```clojure
[:=> [:cat :int] :int]               ; Function: int → int
[:=> [:cat :string :int] :string]    ; Function: (string, int) → string
[:-> :int :int]                      ; Shorthand: int → int
```

### Function with Multiple Arities

```clojure
[:function
 [:=> [:cat :int] :int]              ; Single arg
 [:=> [:cat :int :int] :int]]        ; Two args
```

### Variadic Functions

```clojure
[:=> [:cat :string [:* :int]] :string]  ; String + zero or more ints
[:=> [:cat :int [:+ :string]] :int]     ; Int + one or more strings
```

## Schema Properties

### Common Properties

```clojure
[:string {:min 1
          :max 100
          :error/message "Name must be 1-100 characters"}]

[:int {:min 0
       :max 150
       :error/message "Age must be between 0 and 150"
       :json-schema/example 30}]

[:map {:closed true
       :title "User"
       :description "User entity"}
 [:id uuid?]
 [:name :string]]
```

### Generator Properties

```clojure
[:string {:gen/min 5 :gen/max 10}]   ; Generate 5-10 char strings
[:int {:gen/min 1 :gen/max 100}]     ; Generate ints 1-100
[:vector {:gen/min 2 :gen/max 5} :int]  ; Generate vectors with 2-5 elements
```

### Error Message Properties

```clojure
[:string {:error/message "Custom error message"}]
[:int {:error/message "Must be an integer"
       :error/path [:user :age]}]
```

## Advanced Types

### Every (Lazy Validation)

Validates collections lazily:

```clojure
[:every :int]                        ; Every element is int (lazy)
[:every {:min 1} :string]            ; Every string, min 1 element
```

### Cat (Concatenation)

Sequence of schemas in order:

```clojure
[:cat :string :int :boolean]         ; String, then int, then boolean
[:cat [:* :int]]                     ; Zero or more ints
[:cat :string [:+ :int]]             ; String followed by one or more ints
```

### Catn (Named Cat)

Named sequence elements:

```clojure
[:catn
 [:s :string]
 [:n :int]
 [:b :boolean]]
```

### Alt (Alternative)

One of several alternatives:

```clojure
[:alt
 [:s :string]
 [:n :int]]
```

### Altn (Named Alt)

Named alternatives:

```clojure
[:altn
 [:string :string]
 [:number :int]]
```

### Repeat

Repeated pattern:

```clojure
[:repeat :int]                       ; Repeat int
[:repeat {:min 2 :max 5} :string]    ; 2-5 strings
```

### * (Zero or More)

```clojure
[:* :int]                            ; Zero or more integers
[:* :string]                         ; Zero or more strings
```

### + (One or More)

```clojure
[:+ :int]                            ; One or more integers
[:+ :string]                         ; One or more strings
```

### ? (Optional)

```clojure
[:? :int]                            ; Optional integer
[:? :string]                         ; Optional string
```

## Custom Schema Types

### Registering Custom Types

```clojure
(require '[malli.core :as m])

(def registry
  (merge
    (m/default-schemas)
    {:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]
     :positive-int [:and :int [:> 0]]
     :non-empty-string [:string {:min 1}]}))

; Use with registry
(m/validate :email "test@example.com" {:registry registry})
```

### Using Predicate Functions

```clojure
even?                                ; Built-in predicates work
odd?
empty?
some?
ifn?
```

## Type Coercion Hints

Schemas can include type hints for coercion:

```clojure
[:string]                            ; String type
[:int]                               ; Integer type
[:boolean]                           ; Boolean type
[:keyword]                           ; Keyword type
[:double]                            ; Double type
```

These work with transformers like `mt/string-transformer` to coerce values during decoding.
