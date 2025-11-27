# Recife API Reference

This document provides detailed API reference for the Recife model checker.

## Core Namespace: `recife.core` (aliased as `r`)

### Process Definition

#### `r/defproc`

Creates a process for use in Recife's runtime.

**Signature:**
```clojure
(r/defproc process-name config-or-function)
```

**Usage with function:**
```clojure
(r/defproc tick
  (fn [db]
    (update db ::hour inc)))
```

**Usage with configuration:**
```clojure
(r/defproc ^:fair activate
  {:local {:pc ::try-activation}}
  ...)
```

**Fairness Annotation:**
Mark processes with `^:fair` metadata to guarantee they won't crash unexpectedly. This is required for temporal properties to work correctly, as it prevents stuttering (halting).

```clojure
(r/defproc ^:fair tick-with-fairness
  (fn [db]
    (update db ::hour inc)))
```

**Nil Punning:**
Returning `nil` from a process operator indicates the state remains unchanged. This is useful for creating guards (preconditions):

```clojure
(r/defproc guarded-action
  (fn [db]
    (when (some-condition? db)  ;; Guard: returns nil if false
      (update db :value inc))))
```

### State Transitions

#### `r/goto`

Moves process to a labeled state. Used with local program counter (`:pc`):

```clojure
(r/goto ::next-state)
```

#### `r/done`

Marks process completion. Use this to prevent deadlock violations:

```clojure
(r/done)
```

### Non-determinism

#### `r/one-of`

Declares non-deterministic values from a set. The model checker will explore all possibilities:

```clojure
(def global
  {::paid? (r/one-of #{true false})
   ::amount (r/one-of #{10 20 30})})
```

### Model Execution

#### `r/run-model`

Executes a specification and returns an asynchronous process.

**Signature:**
```clojure
(r/run-model initial-state components & {:keys [trace-example no-deadlock]})
```

**Parameters:**
- `initial-state`: Initial state map
- `components`: Set containing processes and properties to verify
- `:trace-example` (optional): When `true`, returns example trace if no violations found
- `:no-deadlock` (optional): When `true`, disables deadlock detection

**Example:**
```clojure
(def result
  (r/run-model
    {::hour 0}
    #{tick-process hour-property}
    :trace-example true))

;; Dereference to get result
@result
```

**Return Value:**
Returns a future/promise that can be dereferenced with `@` or `deref`.

#### `r/halt!`

Stops the running process and dereferences the result:

```clojure
(r/halt! running-process)
```

#### `r/get-result`

Retrieves the last result from a halted process:

```clojure
(r/get-result)
```

## Helper Namespace: `recife.helpers` (aliased as `rh`)

### Constraints

#### `rh/defconstraint`

Filters states by returning boolean values. Invalid states are discarded.

**Rule:** "Receive db, return boolean; invalid states discarded."

**Example:**
```clojure
(rh/defconstraint disallow-after-25
  [{::keys [hour]}]
  (<= hour 25))
```

**Usage:**
Constraints prune the state space during model checking. States that return `false` are not explored further.

### Invariants

#### `rh/definvariant`

Validates that states satisfy required properties. Violations trigger model-checking failures.

**Rule:** "Return boolean indicating no violation."

**Example:**
```clojure
(rh/definvariant no-hour-after-23
  [{::keys [hour]}]
  (<= hour 23))
```

**Difference from Constraints:**
- **Constraints**: Filter states (pruning)
- **Invariants**: Validate states (failure detection)

### Properties

#### `rh/defproperty`

Declares safety and liveness properties using temporal operators.

**Example:**
```clojure
(rh/defproperty eventually-activated
  [{::keys [status]}]
  (rh/eventually (= status :activated)))
```

### Temporal Operators

#### `rh/eventually`

Asserts a condition will be true at some point during execution (◇P in TLA+).

**Example:**
```clojure
(rh/eventually (= hour 10))
```

**Meaning:** At some state in the trace, `hour` will equal 10.

#### `rh/always`

Validates a condition throughout the entire trace (□P in TLA+).

**Example:**
```clojure
(rh/always (<= hour 23))
```

**Meaning:** In every state of the trace, `hour` will be ≤ 23.

#### `rh/always` + `rh/eventually` (Infinite Recurrence)

Guarantees a state will be reached infinitely often (□◇P in TLA+).

**Example:**
```clojure
(rh/defproperty infinitely-often-10
  [{::keys [hour]}]
  (rh/always (rh/eventually (= hour 10))))
```

**Meaning:** In every suffix of the trace, `hour` will eventually equal 10. This catches scenarios where the system enters limited loops.

**Use case:** Ensure liveness - the system keeps making progress and doesn't get stuck.

#### `rh/eventually` + `rh/always` (Stabilization)

Validates that execution eventually reaches a state and remains there permanently (◇□P in TLA+).

**Example:**
```clojure
(rh/defproperty eventually-stable
  [{::keys [hour]}]
  (rh/eventually (rh/always (= hour 10))))
```

**Meaning:** At some point, `hour` will become 10 and stay 10 forever.

**Use case:** Verify system convergence or quiescence.

## State Management

### State Representation

States are represented as Clojure maps:

```clojure
{::hour 0
 ::minute 0
 ::status :active}
```

### State Modifications

Use standard Clojure functions to modify state:

```clojure
;; Update single value
(update db ::hour inc)

;; Assoc new value
(assoc db ::status :completed)

;; Multiple updates
(-> db
    (update ::hour inc)
    (assoc ::status :ticking))
```

### Guards (Preconditions)

Use `when` to create guarded actions:

```clojure
(r/defproc guarded-increment
  (fn [db]
    (when (< (::hour db) 23)  ;; Guard
      (update db ::hour inc))))
```

If the guard fails, `nil` is returned and the state remains unchanged.

## Traces and Behaviors

### Trace

A **trace** is a sequence of states that the specification progresses through over time:

```
State 0: {::hour 0}
State 1: {::hour 1}
State 2: {::hour 2}
...
```

### Behavior

A **behavior** is a complete execution trace from initial state to terminal state or infinite execution.

### Retrieving Traces

Use `:trace-example true` option with `r/run-model`:

```clojure
(def result
  @(r/run-model
     {::hour 0}
     #{tick}
     :trace-example true))

;; Access trace
(:trace result)
```

## Common Violations

### Deadlock

**What it is:** Occurs when no state advancement is possible and unfinished processes remain.

**How to fix:** Mark processes as complete using `r/done`:

```clojure
(r/defproc terminable-process
  (fn [db]
    (if (completed? db)
      (r/done)  ;; Mark as complete
      (continue-work db))))
```

### Invariant Violation

**What it is:** Triggered when a state fails validation checks defined by invariants.

**How to debug:**
1. Examine the counterexample trace
2. Identify which state violated the invariant
3. Trace back to find which action caused the violation
4. Fix the action's preconditions or the invariant itself

**Example output:**
```
Invariant violation: no-hour-after-23
State 24: {::hour 24}
Action: tick
```

## Complete Example

```clojure
(ns example.clock-spec
  (:require [recife.core :as r]
            [recife.helpers :as rh]))

;; Initial state
(def initial-state
  {::hour 0})

;; Process: increment hour
(r/defproc ^:fair tick
  (fn [db]
    (if (< (::hour db) 23)
      (update db ::hour inc)
      (r/done))))  ;; Complete when hour reaches 23

;; Invariant: hour never exceeds 23
(rh/definvariant hour-in-range
  [{::keys [hour]}]
  (<= 0 hour 23))

;; Property: eventually reaches hour 10
(rh/defproperty eventually-hour-10
  [{::keys [hour]}]
  (rh/eventually (= hour 10)))

;; Run model
(def result
  @(r/run-model
     initial-state
     #{tick hour-in-range eventually-hour-10}
     :trace-example true))

;; Check result
(if (:success? result)
  (println "Verification succeeded!")
  (println "Violation found:" (:error result)))
```

## Comparison to TLA+ Concepts

| Recife | TLA+ | Description |
|--------|------|-------------|
| `r/defproc` | Action | State transition function |
| `rh/definvariant` | Invariant | Safety property (always holds) |
| `rh/defproperty` + `rh/eventually` | Liveness | Eventually property (◇P) |
| `rh/always` | Temporal □ | Always operator |
| `r/one-of` | ∈ (set membership) | Non-deterministic choice |
| `r/done` | Terminal state | Process completion |
| Guard (nil return) | ENABLED | Action precondition |

## Best Practices

1. **Use fairness annotations** (`^:fair`) for processes that should always be enabled
2. **Return nil for guards** instead of throwing exceptions
3. **Mark completion** with `r/done` to avoid deadlock violations
4. **Start simple** with invariants before adding temporal properties
5. **Use constraints** to prune state space early
6. **Name processes descriptively** to make traces readable
7. **Separate concerns** between safety (invariants) and liveness (properties)

## Resources

- Main repository: https://github.com/pfeodrippe/recife
- Documentation: https://recife.pfeodrippe.com/
- Clojars: https://clojars.org/pfeodrippe/recife
