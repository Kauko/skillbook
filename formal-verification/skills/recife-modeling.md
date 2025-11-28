---
name: recife-modeling
description: Use BEFORE implementing concurrent or distributed logic. Creates formal specs in Clojure to find race conditions, deadlocks, and invariant violations BEFORE writing production code.
requires:
  tools: [tlc]
  skills: [tla-concepts]
  deps: [recife/recife]
skip_when:
  - Code is purely sequential with no shared state
  - Already have passing Recife specs and no logic changes
  - User explicitly requests to skip verification (document the risk)
  - Working on UI/presentation layer only
---

# Recife Modeling: TLA+ in Clojure

## Prerequisites

```bash
command -v tlc >/dev/null || { echo "Install: brew install tla-plus"; exit 1; }
grep -q "recife" deps.edn || echo "Add recife dependency - see below"
```

Add if missing:
```clojure
{:deps {recife/recife {:mvn/version "0.1.0"}}}
```

## Quick Start

```clojure
(ns specs.account-transfer
  (:require [recife.core :as r]))

;; Define state variables
(r/defvar balance-a 100)
(r/defvar balance-b 0)

;; Initial state
(r/definit init
  (r/and (= balance-a 100)
         (= balance-b 0)))

;; Actions
(r/defaction transfer [amount]
  (r/and (>= balance-a amount)
         (r/assign balance-a (- balance-a amount))
         (r/assign balance-b (+ balance-b amount))))

;; Invariant (safety property)
(r/definvariant money-conserved
  (= (+ balance-a balance-b) 100))

;; Spec
(r/defspec account-transfer-spec
  {:init init
   :next (r/or (transfer 10) (transfer 20))
   :invariants [money-conserved]})
```

## Run Model Checker

```clojure
(r/run-model account-transfer-spec)
```

**Success**: "Model checking complete. No errors found."

**Violation**: TLC provides counterexample trace - sequence of states leading to violation.

## Gherkin to Recife Translation

| Gherkin | Recife |
|---------|--------|
| **Given** precondition | `r/definit` or action guard |
| **When** action | `r/defaction` with state transitions |
| **Then** postcondition | `r/definvariant` (safety) or `r/defproperty` (liveness) |

## Key Patterns

**Mutual exclusion:**
```clojure
(r/definvariant mutex
  (not (and in-critical-a in-critical-b)))
```

**Eventually responds:**
```clojure
(r/defproperty request-response
  (r/leads-to request-sent response-received))
```

**Bounded value:**
```clojure
(r/definvariant positive-balance
  (>= balance 0))
```

## File Organization

```
src/specs/           ;; Production specs
test/specs/          ;; Exploratory specs
vault/specifications/formal/  ;; Documentation, traces
```

## Reference Documentation

- `references/api.md` - Complete API reference
- `references/examples.md` - Five runnable examples
- `references/tlc-integration.md` - TLC integration and result interpretation

## Success Criteria

```bash
#!/bin/bash
# verify-recife.sh - Machine-readable verification

verify_recife() {
  local spec_ns="${1:-specs.main}"
  local nrepl_port="${2:-7888}"

  echo "recife_verification:start"
  echo "spec_namespace:$spec_ns"

  # Check TLC installed
  if command -v tlc &>/dev/null; then
    echo "tlc:installed"
  else
    echo "tlc:missing"
    echo "recife_verification:exit_code:1"
    return 1
  fi

  # Check nREPL available
  if ! command -v clj-nrepl-eval &>/dev/null; then
    echo "nrepl_eval:missing"
    echo "recife_verification:exit_code:1"
    return 1
  fi

  # Try to load and run spec
  local result
  result=$(clj-nrepl-eval -p "$nrepl_port" \
    "(require '[$spec_ns]) (${spec_ns}/run-verification)" 2>&1)

  if echo "$result" | grep -q "No errors found"; then
    echo "model_check:pass"
    echo "recife_verification:exit_code:0"
    return 0
  elif echo "$result" | grep -q "Error"; then
    echo "model_check:fail"
    echo "counterexample:$(echo "$result" | grep -A5 'State')"
    echo "recife_verification:exit_code:1"
    return 1
  else
    echo "model_check:unknown"
    echo "output:$result"
    echo "recife_verification:exit_code:2"
    return 2
  fi
}

verify_recife "$@"
```

**For REPL-based verification:**
```clojure
;; In REPL - machine-readable output
(defn run-verification []
  (let [result (r/run-model my-spec)]
    (if (:ok result)
      (println "model_check:pass")
      (do
        (println "model_check:fail")
        (println "states_explored:" (:states result))
        (println "counterexample:" (:trace result))))))
```

**Success = one of:**
- [ ] `model_check:pass` - All invariants hold, no deadlocks
- [ ] `model_check:fail` with `counterexample:` - Violation found and understood

## Related Skills

- `tla-concepts` - TLA+ fundamentals (state machines, temporal logic)
