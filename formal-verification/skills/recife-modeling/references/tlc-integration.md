# TLC Integration Guide

This document explains how Recife integrates with TLC (TLA+ model checker), how to run model checking, and how to interpret results.

## What is TLC?

TLC is the model checker for TLA+ (Temporal Logic of Actions). Recife uses TLC as its underlying verification engine, translating Clojure specifications into TLA+ specifications that TLC can verify.

## Architecture

```
Clojure Spec (Recife)
        ↓
    Translation
        ↓
   TLA+ Spec
        ↓
   TLC Engine
        ↓
 Verification Result
        ↓
  Clojure Result
```

Recife handles the translation and result interpretation automatically.

## Running Model Checking

### Basic Execution

```clojure
(require '[recife.core :as r]
         '[recife.helpers :as rh])

(def result
  @(r/run-model
     initial-state
     #{processes invariants properties}))

(if (:success? result)
  (println "Verification passed!")
  (println "Violation found!"))
```

### With Trace Output

```clojure
(def result
  @(r/run-model
     initial-state
     #{processes invariants properties}
     :trace-example true))

;; Access the trace
(:trace result)
```

### Without Deadlock Detection

Some specifications intentionally have terminal states. Disable deadlock checking:

```clojure
(def result
  @(r/run-model
     initial-state
     #{processes invariants properties}
     :no-deadlock true))
```

### Asynchronous Execution

`r/run-model` returns a future that can be dereferenced:

```clojure
;; Start verification (non-blocking)
(def running-model
  (r/run-model initial-state components))

;; Do other work...

;; Wait for result
@running-model

;; Or halt explicitly
(r/halt! running-model)
```

## Result Structure

### Success Result

```clojure
{:success? true
 :states-explored 1523
 :trace [...]}  ;; If :trace-example true
```

### Violation Result

```clojure
{:success? false
 :error "Invariant violation: money-conserved"
 :counterexample [{...} {...} ...]  ;; Trace leading to violation
 :states-explored 847}
```

## Interpreting Results

### States Explored

The `:states-explored` field tells you how much of the state space was examined:

```clojure
(:states-explored result)
;; => 1523
```

**What it means:**
- Small number (< 100): Simple spec or heavily constrained
- Medium (100-10,000): Typical specification
- Large (> 10,000): Complex spec with large state space
- Very large (> 100,000): May need optimization (see State Space Reduction below)

### Verification Success

```clojure
(if (:success? result)
  (println "All invariants and properties satisfied!")
  (println "Violation found"))
```

**When success?**
- All invariants hold in every explored state
- All temporal properties are satisfied
- No deadlocks detected (unless `:no-deadlock true`)

### Understanding Traces

A trace is a sequence of states:

```clojure
(:trace result)
;; => [{::hour 0} {::hour 1} {::hour 2} ...]
```

**Analyzing traces:**

```clojure
;; Count steps
(count (:trace result))

;; View first state
(first (:trace result))

;; View last state
(last (:trace result))

;; Extract specific field across trace
(map ::hour (:trace result))
;; => (0 1 2 3 4 5 ...)

;; View state transitions
(doseq [[idx state] (map-indexed vector (:trace result))]
  (println "Step" idx ":" state))
```

## Counterexamples

When TLC finds a violation, it produces a **counterexample**: a trace from initial state to the violating state.

### Counterexample Structure

```clojure
{:success? false
 :error "Invariant violation: money-conserved"
 :counterexample
 [{::account-a 100 ::account-b 50 ::total 150}   ;; State 0 (initial)
  {::account-a 70 ::account-b 80 ::total 150}    ;; State 1 (after action)
  {::account-a 70 ::account-b 110 ::total 150}]}  ;; State 2 (VIOLATION!)
```

### Reading Counterexamples

**Step-by-step:**

1. **Identify violation point**
   ```clojure
   (count (:counterexample result))
   ;; => 3 (violation at state 2)
   ```

2. **Examine violating state**
   ```clojure
   (last (:counterexample result))
   ;; => {::account-a 70 ::account-b 110 ::total 150}
   ;; Expected: 70 + 110 = 180, but total says 150!
   ```

3. **Trace back to understand cause**
   ```clojure
   (doseq [[idx state] (map-indexed vector (:counterexample result))]
     (println "State" idx ":"
              "A =" (::account-a state)
              "B =" (::account-b state)
              "Total =" (::total state)
              "Sum =" (+ (::account-a state) (::account-b state))))
   ```

4. **Compute diffs between states**
   ```clojure
   (defn state-diff [s1 s2]
     (into {}
       (for [k (keys s1)
             :when (not= (get s1 k) (get s2 k))]
         [k {:before (get s1 k)
             :after (get s2 k)}])))

   (let [states (:counterexample result)]
     (doseq [[s1 s2] (partition 2 1 states)]
       (println "Changed:" (state-diff s1 s2))))
   ```

### Common Violation Types

#### Invariant Violation

**What happened:** A state violated a safety property.

**Example error:**
```
:error "Invariant violation: money-conserved"
```

**How to debug:**
1. Identify which invariant failed
2. Look at the violating state
3. Trace back to find which action caused it
4. Fix the action or the invariant

**Fix strategies:**
- Add guard to action (prevent invalid transitions)
- Fix action logic (correct the state transformation)
- Relax invariant (if it's too strict)

#### Deadlock Violation

**What happened:** No actions are enabled and processes are not complete.

**Example error:**
```
:error "Deadlock detected"
```

**How to debug:**
1. Look at final state in counterexample
2. Check why no actions are enabled
3. Verify process completion logic

**Fix strategies:**
- Add `r/done` to mark completion
- Fix action guards (may be too restrictive)
- Use `:no-deadlock true` if terminal states are expected

#### Temporal Property Violation

**What happened:** A liveness property was not satisfied.

**Example error:**
```
:error "Temporal property violation: eventually-activated"
```

**How to debug:**
1. Check if actions have `^:fair` metadata
2. Verify the property is achievable
3. Look for infinite loops that avoid the desired state

**Fix strategies:**
- Add `^:fair` to actions
- Fix action logic to enable progress
- Adjust temporal property (may be too strong)

## State Space Reduction

If TLC explores too many states or runs too long, use these techniques:

### 1. Add Constraints

Prune states early:

```clojure
(rh/defconstraint reasonable-bounds
  [db]
  (and (<= (::counter db) 100)
       (<= (count (::queue db)) 50)))
```

### 2. Reduce Non-Determinism

Instead of:
```clojure
{::amount (r/one-of #{10 20 30 40 50})}
```

Use representative values:
```clojure
{::amount (r/one-of #{10 50})}  ;; Just min and max
```

### 3. Limit Sequence Lengths

Bound collections:

```clojure
(defn bounded-conj [coll item max-size]
  (if (< (count coll) max-size)
    (conj coll item)
    coll))
```

### 4. Use Symmetry (TLA+ Feature)

If your spec has symmetric components (e.g., identical processes), TLC can exploit this. Recife may support symmetry reduction - check documentation.

### 5. Model Smaller System

Reduce the number of processes or resources:

```clojure
;; Instead of 10 processes
(def num-processes 3)  ;; Test with 3 first
```

## TLC Configuration

While Recife abstracts TLC configuration, understanding TLC's capabilities helps:

### State Space Limits

TLC explores states until:
- All reachable states are explored (finite state space)
- A violation is found
- A resource limit is hit (memory, time)

### Deadlock Detection

TLC considers a state deadlocked if:
- No next state is possible
- Some process is not marked complete

### Fairness

Recife's `^:fair` annotation tells TLC that an action is **weakly fair**: if an action is eventually always enabled, it will eventually be taken.

This is critical for liveness properties.

## Debugging Tips

### 1. Start with Invariants Only

Before adding properties, verify safety:

```clojure
@(r/run-model
   initial-state
   #{actions invariants}  ;; No properties yet
   :trace-example true)
```

### 2. Use Small State Space Initially

Test with minimal non-determinism:

```clojure
;; Start small
{::value (r/one-of #{0 1})}

;; Expand later
{::value (r/one-of #{0 1 2 3 4 5})}
```

### 3. Add Temporary Invariants

To verify reachability:

```clojure
(rh/definvariant should-fail-if-reachable
  "Temporary: verify we reach this state"
  [{::keys [status]}]
  (not= status :completed))
```

If this fails, you know `:completed` is reachable.

### 4. Inspect Traces Manually

```clojure
(def result @(r/run-model ...))

;; Print readable trace
(doseq [[idx state] (map-indexed vector (:trace result))]
  (println)
  (println "==== State" idx "====")
  (clojure.pprint/pprint state))
```

### 5. Use `println` in Processes (Carefully)

For debugging, you can add logging:

```clojure
(r/defproc debug-action
  (fn [db]
    (println "Current state:" db)
    (update db ::counter inc)))
```

**Warning:** This will be called during translation, which may be confusing. Use sparingly.

## Performance Tips

### Fast Verification

1. **Small state space**: Use constraints aggressively
2. **Few temporal properties**: Start with invariants
3. **Limited non-determinism**: Use representative values
4. **Short traces**: Bound sequences and counters

### When TLC is Slow

1. **Check states explored**: Is it reasonable?
   ```clojure
   (:states-explored result)
   ```

2. **Profile state space growth**: Add constraints incrementally

3. **Simplify specification**: Remove features temporarily

4. **Verify incrementally**: Add actions one at a time

## Advanced: TLA+ Output

Recife generates TLA+ specifications behind the scenes. While you don't usually need to see this, it can be useful for debugging or understanding.

**TLC concepts mapped to Recife:**

| TLC Concept | Recife Equivalent |
|------------|------------------|
| VARIABLES | State map keys |
| Init | Initial state with `r/one-of` expanded |
| Next | All processes (non-deterministic choice) |
| Invariant | `rh/definvariant` |
| Property | `rh/defproperty` |
| ENABLED | Guard (returns non-nil) |
| Fairness | `^:fair` metadata |

## Error Messages

### Common Errors

**"Deadlock detected"**
- Add `r/done` to processes
- Or use `:no-deadlock true`

**"Invariant violation: <name>"**
- Check counterexample
- Trace back to violating action
- Fix action or invariant

**"Temporal property violation: <name>"**
- Add `^:fair` to actions
- Verify property is achievable

**"State space explosion"**
- Add constraints
- Reduce non-determinism
- Bound sequences

## Best Practices

1. **Start simple**: Verify basic invariants before adding properties
2. **Use constraints early**: Prune state space at the source
3. **Test with small models**: Scale up after verification passes
4. **Examine traces**: Always look at example traces, even on success
5. **Iterate**: Spec → Run → Fix → Repeat
6. **Save results**: Document successful verifications and violations
7. **Fairness matters**: Always add `^:fair` for temporal properties

## Resources

### TLA+ Learning

Understanding TLA+ concepts helps use Recife effectively:
- [Learn TLA+](https://learntla.com/)
- [TLA+ Home](https://lamport.azurewebsites.net/tla/tla.html)
- [TLC Manual](https://tla.msr-inria.inria.fr/tlatoolbox/doc/model/executing-tlc.html)

### Recife Resources

- GitHub: https://github.com/pfeodrippe/recife
- Documentation: https://recife.pfeodrippe.com/
- Clojars: https://clojars.org/pfeodrippe/recife

### Related Concepts

- [Model Based Techniques](https://mbt.informal.systems/docs/tla_basics_tutorials/hello_world.html)
- [TLA+ Specifications](https://old.learntla.com/introduction/example/)
- [Medium: Introduction to TLA+ Model Checking](https://medium.com/software-safety/introduction-to-tla-model-checking-in-the-command-line-c6871700a6a2)

## Summary

Recife abstracts TLC complexity while providing full access to its verification power:

- **Run models** with `r/run-model`
- **Check success** with `:success?` field
- **Examine traces** to understand behavior
- **Debug violations** with counterexamples
- **Optimize** with constraints and reduced non-determinism
- **Iterate** until verification passes

The key to effective model checking is starting simple and iterating based on results.
