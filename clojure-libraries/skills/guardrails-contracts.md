---
name: guardrails-contracts
description: Use when writing functions at system boundaries (API handlers, public namespace interfaces). Add contracts with >defn to validate inputs/outputs during development.
requires:
  tools: []
  skills: [malli-schemas]
  deps: [com.fulcrologic/guardrails, metosin/malli]
skip_when:
  - Internal helper functions with trusted callers
  - Performance-critical hot paths (contracts add overhead in dev)
  - Project doesn't use guardrails (check deps.edn)
---

# Guardrails Runtime Contracts

## Prerequisites

```bash
grep -q "guardrails" deps.edn || echo "Add guardrails dependency - see below"
```

Add if missing:
```clojure
{:deps {com.fulcrologic/guardrails {:mvn/version "1.1.10"}
        metosin/malli {:mvn/version "0.17.0"}}}
```

## Configuration

`guardrails.edn`:
```clojure
{:defn-macro true      ; Enable >defn macro
 :throw? true          ; Throw on violation
 :emit-spec false}     ; Don't emit clojure.spec
```

## Basic Usage

```clojure
(ns myapp.core
  (:require [com.fulcrologic.guardrails.core :refer [>defn =>]]
            [malli.core :as m]))

;; With Malli schema
(>defn create-user
  [name email]
  [string? [:re #".+@.+"] => [:map [:id uuid?] [:name string?] [:email string?]]]
  {:id (random-uuid)
   :name name
   :email email})

;; Multiple arities
(>defn greet
  ([name]
   [string? => string?]
   (str "Hello, " name))
  ([first-name last-name]
   [string? string? => string?]
   (str "Hello, " first-name " " last-name)))
```

## Contract Syntax

```clojure
;; Basic: [input-specs => output-spec]
[string? int? => boolean?]

;; With Malli schemas
[[:string {:min 1}] int? => [:map [:result any?]]]

;; Variadic
[string? & [int?] => string?]

;; Any input, specific output
[any? any? => [:enum :ok :error]]
```

## Production Config

Disable in production (no overhead):
```clojure
;; guardrails.edn
{:defn-macro false
 :throw? false}
```

Or via environment:
```bash
GUARDRAILS=false clojure -M:run
```

## Key Patterns

**Boundary validation:**
```clojure
;; Validate at API boundaries, trust internal code
(>defn api-handler
  [request]
  [Request => Response]
  (internal-logic (:body request)))
```

**Optional contract checking:**
```clojure
;; Use >defn in dev, regular defn in prod
#?(:clj (>defn foo [x] [int? => int?] (inc x))
   :cljs (defn foo [x] (inc x)))
```

## Reference Documentation

- `references/syntax.md` - Complete >defn syntax
- `references/configuration.md` - All config options
- `references/analyzer.md` - Static analysis setup

## Success Criteria

- [ ] `guardrails.edn` configuration exists
- [ ] Functions use `>defn` with contract specs
- [ ] Contract violations throw in dev, are silent in prod

## Related Skills

- `malli-schemas` - Schema definitions
