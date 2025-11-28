---
name: malli-schemas
description: Use when defining data structures that cross boundaries (API input/output, database records, events). Define schemas FIRST, then implement code that uses them.
requires:
  tools: []
  skills: []
  deps: [metosin/malli]
skip_when:
  - Data is purely internal and short-lived (local let bindings)
  - Working with existing schemas that don't need changes
  - User asks for spec instead of malli
---

# Malli Schema Patterns

## Prerequisites

```bash
grep -q "metosin/malli" deps.edn || echo "Add malli dependency - see below"
```

Add if missing:
```clojure
{:deps {metosin/malli {:mvn/version "0.17.0"}}}
```

## Quick Reference

### Common Types

```clojure
;; Primitives
int? string? boolean? keyword? uuid? inst?

;; Constrained
[:string {:min 1 :max 100}]
[:int {:min 0 :max 150}]
[:re #"^[a-z]+$"]

;; Collections
[:vector int?]
[:set keyword?]
[:map [:id uuid?] [:name string?]]
[:map-of keyword? any?]

;; Optional/nullable
[:maybe string?]
[:map [:email {:optional true} string?]]

;; Unions and enums
[:or string? int?]
[:enum "a" "b" "c"]

;; Discriminated unions
[:multi {:dispatch :type}
 [:user/created [:map [:type [:= :user/created]] [:user-id uuid?]]]
 [:user/deleted [:map [:type [:= :user/deleted]] [:user-id uuid?]]]]
```

### Validation

```clojure
(require '[malli.core :as m] '[malli.error :as me])

(m/validate [:string {:min 1}] "hello")  ;; => true

(-> (m/explain [:int {:max 10}] 15)
    (me/humanize))  ;; => ["should be at most 10"]
```

### Transformation

```clojure
(require '[malli.transform :as mt])

;; String coercion (from JSON/HTTP params)
(m/decode [:map [:age int?]]
          {:age "30"}
          (mt/string-transformer))
;; => {:age 30}

;; Strip extra keys
(m/decode schema data (mt/strip-extra-keys-transformer))

;; Apply defaults
(m/decode [:map [:count {:default 0} int?]]
          {}
          (mt/default-value-transformer))
;; => {:count 0}
```

### Generation

```clojure
(require '[malli.generator :as mg])

(mg/generate [:map [:id uuid?] [:name string?]])
;; => {:id #uuid "..." :name "abc"}

(mg/sample schema 10)  ;; 10 samples
```

### Function Schemas

```clojure
(require '[malli.instrument :as mi])

(defn create-user [data] ...)

(m/=> create-user [:=> [:cat :map] :map])

(mi/instrument!)  ;; Enable validation in dev
```

## Key Patterns

**Entity schema:**
```clojure
(def User
  [:map {:closed true}
   [:id uuid?]
   [:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]
   [:name [:string {:min 1 :max 100}]]])
```

**Registry for reuse:**
```clojure
(def registry
  (merge (m/default-schemas)
         {:user/email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]
          ::user [:map [:email :user/email]]}))
```

**Custom error messages:**
```clojure
[:string {:min 8 :error/message "Password must be at least 8 characters"}]
```

## Success Criteria

- [ ] Schemas validate expected data correctly
- [ ] Invalid data produces humanized error messages
- [ ] Transformers coerce data as expected (if used)

## Reference Documentation

- `references/schema-types.md` - Complete type reference
- `references/transformations.md` - Coercion and encoding
- `references/generation.md` - Property-based testing
- `references/function-schemas.md` - Function instrumentation
