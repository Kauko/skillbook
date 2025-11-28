---
name: property-testing
description: Use when you have Malli schemas and want to verify functions behave correctly for ALL valid inputs, not just example cases. Run after writing schemas and functions.
requires:
  tools: []
  skills: [malli-schemas]
  deps: [metosin/malli, org.clojure/test.check]
skip_when:
  - No Malli schemas defined for the data being tested
  - Function is trivial (pure data transformation with no edge cases)
  - Already have comprehensive example-based tests and no schema changes
  - Performance-critical code where generator overhead is prohibitive
---

# Property-Based Testing with Malli

Generate hundreds of test cases automatically from your Malli schemas.

## Prerequisites

```bash
grep -q "test.check" deps.edn || echo "Add test.check dependency"
grep -q "metosin/malli" deps.edn || echo "Add malli dependency"
```

Add if missing:
```clojure
{:deps {metosin/malli {:mvn/version "0.17.0"}
        org.clojure/test.check {:mvn/version "1.1.1"}}}
```

## Core Concept

Instead of writing:
```clojure
;; Example-based: tests 3 cases
(deftest add-test
  (is (= 3 (add 1 2)))
  (is (= 0 (add 0 0)))
  (is (= -1 (add 1 -2))))
```

Write:
```clojure
;; Property-based: tests 100+ generated cases
(defspec add-commutative 100
  (for-all [a (mg/generator :int)
            b (mg/generator :int)]
    (= (add a b) (add b a))))
```

## Workflow

### 1. Identify Properties

Properties are invariants that should hold for ALL valid inputs:

| Property Type | Example |
|--------------|---------|
| **Commutative** | `(f a b) = (f b a)` |
| **Associative** | `(f (f a b) c) = (f a (f b c))` |
| **Identity** | `(f x identity-element) = x` |
| **Idempotent** | `(f (f x)) = (f x)` |
| **Round-trip** | `(decode (encode x)) = x` |
| **Invariant preserved** | `(valid? (transform x))` when `(valid? x)` |

### 2. Write Property Tests

```clojure
(ns myapp.core-test
  (:require [clojure.test :refer [deftest is]]
            [clojure.test.check.clojure-test :refer [defspec]]
            [clojure.test.check.properties :refer [for-all]]
            [malli.generator :as mg]))

;; Use schema from production code
(def User
  [:map
   [:id uuid?]
   [:name [:string {:min 1 :max 100}]]
   [:email [:re #"^[^\s@]+@[^\s@]+\.[^\s@]+$"]]])

;; Property: serialization round-trips
(defspec user-serialization-roundtrip 100
  (for-all [user (mg/generator User)]
    (= user (-> user serialize deserialize))))

;; Property: validation is consistent
(defspec valid-users-stay-valid 100
  (for-all [user (mg/generator User)]
    (valid-user? user)))
```

### 3. Test Function Schemas

Malli can automatically test functions against their schemas:

```clojure
(require '[malli.core :as m]
         '[malli.generator :as mg])

;; Define function with schema
(defn process-user [user]
  {:pre [(m/validate User user)]}
  (update user :name str/upper-case))

;; Schema for the function itself
(def ProcessUser
  [:=> [:cat User] User])

;; Test: function satisfies its contract
(defspec process-user-contract 100
  (for-all [user (mg/generator User)]
    (m/validate User (process-user user))))
```

### 4. Handle Failures

When a property fails, test.check **shrinks** to find the minimal failing case:

```
FAIL in (user-serialization-roundtrip)
Falsifiable after 23 tries, shrunk to:
{:id #uuid "00000000-0000-0000-0000-000000000000"
 :name ""
 :email "a@b.c"}
```

This tells you the **simplest** input that breaks your property.

## Common Patterns

### Constrained Generators

```clojure
;; Generate only positive integers
(mg/generator [:int {:min 1}])

;; Generate non-empty strings
(mg/generator [:string {:min 1}])

;; Generate from enum
(mg/generator [:enum :pending :active :completed])
```

### Custom Generators

```clojure
(require '[clojure.test.check.generators :as gen])

;; When default generator doesn't work
(def custom-schema
  [:string {:gen/gen (gen/fmap str/upper-case gen/string-alphanumeric)}])
```

### Testing State Transitions

```clojure
(def OrderState [:enum :draft :submitted :paid :shipped])

(defspec valid-state-transitions 100
  (for-all [from-state (mg/generator OrderState)
            to-state (mg/generator OrderState)]
    (if (valid-transition? from-state to-state)
      (= to-state (transition from-state to-state))
      (thrown? ExceptionInfo (transition from-state to-state)))))
```

## Integration with clojure.test

```clojure
;; Run with standard test runner
;; lein test
;; clojure -M:test
```

Property tests run alongside regular tests. Failures show:
- Number of tests run before failure
- The failing input
- The shrunk (minimal) failing input

## Success Criteria

```bash
#!/bin/bash
# verify-property-tests.sh

verify_property_tests() {
  echo "property_test_verification:start"

  # Check dependencies
  if ! grep -q "test.check" deps.edn 2>/dev/null; then
    echo "test_check_dep:missing"
    echo "property_test_verification:exit_code:1"
    return 1
  fi
  echo "test_check_dep:present"

  # Check for defspec usage
  local defspec_count
  defspec_count=$(grep -r "defspec" test/ --include="*.clj" 2>/dev/null | wc -l | tr -d ' ')
  echo "defspec_count:$defspec_count"

  if [ "$defspec_count" -eq 0 ]; then
    echo "property_tests:none_found"
    echo "property_test_verification:exit_code:1"
    return 1
  fi

  # Run tests
  if clojure -M:test 2>&1 | tee /tmp/test-output.txt | grep -q "FAIL\|ERROR"; then
    echo "test_run:fail"
    echo "property_test_verification:exit_code:1"
    return 1
  fi

  echo "test_run:pass"
  echo "property_test_verification:exit_code:0"
  return 0
}

verify_property_tests
```

**Success = all checks pass:**
- [ ] `test_check_dep:present` - test.check in deps.edn
- [ ] `defspec_count:>0` - At least one property test exists
- [ ] `test_run:pass` - All property tests pass

## Reference

- [test.check beginner guide](https://clojure.org/guides/test_check_beginner)
- [Malli generators](https://github.com/metosin/malli#value-generation)
- [Lambda Island tutorial](https://lambdaisland.com/episodes/generative-testing-clojure-test-check)

## Related Skills

- `malli-schemas` - Define schemas that generators use
- `guardrails-contracts` - Runtime validation (complementary to property tests)
