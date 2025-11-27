---
description: Use when creating formal specifications in Clojure using Recife. Translates Gherkin scenarios to TLA+ predicates, runs TLC model checker, and interprets results.
---

# Recife Modeling: TLA+ in Clojure

## Overview

Recife is a Clojure library that brings TLA+ model checking to Clojure. Instead of learning TLA+'s syntax, write specifications as Clojure data structures and functions.

**Reference Documentation:** This skill includes comprehensive reference materials in the `references/` directory:
- [`references/api.md`](references/api.md) - Complete API reference for all Recife functions and macros
- [`references/examples.md`](references/examples.md) - Five complete, runnable specification examples
- [`references/tlc-integration.md`](references/tlc-integration.md) - TLC integration, running models, interpreting results

This main document focuses on **workflow and translation patterns**. For detailed API information, see the reference files.

**Key Benefits**:
- Write specs in familiar Clojure syntax
- Leverage Clojure's data manipulation capabilities
- Integrate formal verification into your Clojure workflow
- Generate TLA+ and run TLC model checker automatically

**Use this skill when**:
- Creating formal specifications for concurrent/distributed systems
- Translating Gherkin scenarios to formal models
- Verifying correctness properties before implementation
- Investigating subtle race conditions or edge cases

## Prerequisites Check

Before starting, verify Recife is in your project dependencies:

<recife-setup>
1. Check `deps.edn` for Recife:
   ```bash
   grep -A 2 "recife" deps.edn
   ```

2. If not present, add to `deps.edn`:
   ```clojure
   {:deps {recife/recife {:mvn/version "0.1.0"}}}  ;; Check for latest version
   ```

3. Verify TLC (TLA+ model checker) is available:
   ```bash
   which tlc
   ```

   If not installed, install via:
   - Mac: `brew install tla-plus`
   - Linux: Download from https://github.com/tlaplus/tlaplus/releases
   - Ensure `tlc` command is in PATH

4. Create spec directories if they don't exist:
   ```bash
   mkdir -p src/specs test/specs vault/specifications/formal
   ```
</recife-setup>

## File Organization

**Specification placement**:

- **Exploratory specs**: `test/specs/` (experimenting, learning)
- **Production specs**: `src/specs/` (verified designs, maintained specs)
- **Documentation**: `vault/specifications/formal/` (traces, notes, ADRs)

**Naming convention**:
```
src/specs/
├── account_transfer_spec.clj       (spec file)
└── account_transfer_spec_test.clj  (test running TLC)

vault/specifications/formal/
├── account-transfer-spec.md        (human-readable doc)
├── account-transfer-trace.txt      (counterexample if found)
└── account-transfer-adr.md         (design decisions)
```

## Starter Spec Template

Create a new spec using this template:

```clojure
(ns specs.my-system-spec
  "Formal specification for [SYSTEM DESCRIPTION]

   Verifies:
   - [SAFETY PROPERTY 1]
   - [SAFETY PROPERTY 2]
   - [LIVENESS PROPERTY]"
  (:require [recife.core :as r]
            [recife.helpers :as rh]))

;; ====================
;; State Space
;; ====================

(def initial-state
  "Defines all possible initial states"
  {::variable-1 0
   ::variable-2 (r/one-of #{"possible" "values"})
   ::variable-3 []})

;; ====================
;; Processes (Actions)
;; ====================

(r/defproc ^:fair action-1
  "Description of what this action does

   Preconditions:
   - [CONDITION 1]
   - [CONDITION 2]

   Effects:
   - [CHANGE 1]
   - [CHANGE 2]"
  (fn [db]
    (when (precondition? db)  ;; Guard: returns nil if false
      (-> db
          (update ::variable-1 inc)
          (update ::variable-2 conj "new-value")))))

(r/defproc ^:fair action-2
  "Description of what this action does"
  (fn [db]
    (when (another-precondition? db)
      (assoc db ::variable-3 (compute-next-value db)))))

(r/defproc ^:fair complete
  "Mark system as complete"
  (fn [db]
    (when (done? db)
      (r/done))))

;; ====================
;; Invariants
;; ====================

(rh/definvariant type-ok
  "Type invariant: variables have expected types/ranges"
  [db]
  (and (nat-int? (::variable-1 db))
       (string? (::variable-2 db))
       (vector? (::variable-3 db))))

(rh/definvariant business-rule-ok
  "Business logic invariant: [DESCRIBE RULE]"
  [db]
  (implies (::condition db)
           (::expected-property db)))

;; ====================
;; Properties
;; ====================

(rh/defproperty eventually-completed
  "Eventually property: [DESCRIBE WHAT EVENTUALLY HAPPENS]"
  [db]
  (rh/eventually (= (::status db) :completed)))

;; ====================
;; Model Execution
;; ====================

(defn run-spec []
  "Run the model checker"
  @(r/run-model
     initial-state
     #{action-1
       action-2
       complete
       type-ok
       business-rule-ok
       eventually-completed}
     :trace-example true))

;; ====================
;; Helpers
;; ====================

(defn implies [p q]
  "Logical implication: p => q"
  (or (not p) q))

(defn precondition? [db]
  "Check if action preconditions are satisfied"
  (and (::some-flag db)
       (>= (::variable-1 db) 10)))

(defn another-precondition? [db]
  (some? (::variable-2 db)))

(defn compute-next-value [db]
  [(::variable-1 db)])

(defn done? [db]
  (= (::status db) :completed))

;; ====================
;; Testing
;; ====================

(comment
  ;; Run verification
  (def result (run-spec))

  ;; Check success
  (:success? result)

  ;; View trace
  (count (:trace result))
  (first (:trace result))

  ;; Debug violation
  (when-not (:success? result)
    (println "Error:" (:error result))
    (clojure.pprint/pprint (:counterexample result)))
  )
```

## Gherkin to Recife Translation

### Translation Process

**Step 1**: Read Gherkin scenarios from `test/features/`

**Step 2**: Identify TLA+ elements:
- **Given** clauses → Initial state or action preconditions
- **When** clauses → Actions (state transitions)
- **Then** clauses → Invariants or assertions

**Step 3**: Translate to Recife Clojure code

**Step 4**: Add invariants that must hold across ALL behaviors

### Example Translation

**Gherkin** (`test/features/account_transfer.feature`):
```gherkin
Feature: Account Transfer

  Scenario: Successful transfer with sufficient balance
    Given account A has balance 100
    And account B has balance 50
    When 30 is transferred from A to B
    Then account A should have balance 70
    And account B should have balance 80
    And total money should be conserved

  Scenario: Transfer fails with insufficient balance
    Given account A has balance 20
    And account B has balance 50
    When 30 is transferred from A to B
    Then the transfer should fail
    And balances should remain unchanged
```

**Recife Spec** (`src/specs/account_transfer_spec.clj`):

```clojure
(ns specs.account-transfer-spec
  "Formal specification for account transfer system

   Verifies:
   - Money is always conserved (safety)
   - Balances never go negative (safety)
   - Transfers with sufficient balance eventually succeed (liveness)"
  (:require [recife.core :as r]
            [recife.helpers :as rh]))

;; ====================
;; State Space
;; ====================

(def initial-state
  "Initial state space: various starting balances"
  {::balance-a (r/one-of #{50 100})
   ::balance-b (r/one-of #{50 100})
   ::total nil
   ::last-transfer nil})

(defn init-with-total [state]
  "Initialize with consistent total"
  (assoc state ::total (+ (::balance-a state) (::balance-b state))))

;; ====================
;; Processes
;; ====================

(r/defproc ^:fair transfer-a-to-b
  "Transfer 30 from account A to B"
  (fn [db]
    (when (>= (::balance-a db) 30)  ;; Guard: sufficient balance
      (-> db
          (update ::balance-a - 30)
          (update ::balance-b + 30)
          (assoc ::last-transfer {:from :a :to :b :amount 30})))))

(r/defproc ^:fair transfer-b-to-a
  "Transfer 30 from account B to A"
  (fn [db]
    (when (>= (::balance-b db) 30)
      (-> db
          (update ::balance-b - 30)
          (update ::balance-a + 30)
          (assoc ::last-transfer {:from :b :to :a :amount 30})))))

(r/defproc ^:fair complete
  "Complete after at least one transfer"
  (fn [db]
    (when (::last-transfer db)
      (r/done))))

;; ====================
;; Invariants
;; ====================

(rh/definvariant type-ok
  "Type invariant: balances are natural numbers"
  [db]
  (and (nat-int? (::balance-a db))
       (nat-int? (::balance-b db))
       (nat-int? (::total db))))

(rh/definvariant money-conserved
  "Safety invariant: total money is always conserved"
  [db]
  (= (+ (::balance-a db) (::balance-b db))
     (::total db)))

(rh/definvariant no-negative-balance
  "Safety invariant: balances never go negative"
  [db]
  (and (>= (::balance-a db) 0)
       (>= (::balance-b db) 0)))

;; ====================
;; Properties
;; ====================

(rh/defproperty transfer-eventually-happens
  "At least one transfer eventually happens"
  [db]
  (rh/eventually (some? (::last-transfer db))))

;; ====================
;; Model Execution
;; ====================

(defn run-transfer-spec []
  "Run the model checker"
  @(r/run-model
     (init-with-total initial-state)
     #{transfer-a-to-b
       transfer-b-to-a
       complete
       type-ok
       money-conserved
       no-negative-balance
       transfer-eventually-happens}
     :trace-example true))

;; ====================
;; Testing
;; ====================

(comment
  ;; Run verification
  (def result (run-transfer-spec))

  ;; Check success
  (:success? result)

  ;; View conservation across trace
  (doseq [state (:trace result)]
    (println "A:" (::balance-a state)
             "B:" (::balance-b state)
             "Total:" (::total state)
             "Sum:" (+ (::balance-a state) (::balance-b state))))
  )
```

### Running the Model Checker

```bash
# Run verification from REPL
clj
user=> (require 'specs.account-transfer-spec)
user=> (def result (specs.account-transfer-spec/run-transfer-spec))
user=> (:success? result)

# Or run as script
clj -M -e "(require 'specs.account-transfer-spec) (clojure.pprint/pprint (specs.account-transfer-spec/run-transfer-spec))"

# Save results
clj -M -e "(require 'specs.account-transfer-spec) (clojure.pprint/pprint (specs.account-transfer-spec/run-transfer-spec))" \
  | tee vault/specifications/formal/account-transfer-results.txt
```

## Interpreting Counterexamples

When TLC finds an invariant violation, it produces a counterexample trace.

### Reading Counterexample Output

**Example Output**:
```
Invariant violation: money-conserved

State 1 (Initial):
{:balance-a 100
 :balance-b 50
 :total 150
 :last-transfer nil}

State 2 (transfer-a-to-b 30):
{:balance-a 70
 :balance-b 80
 :total 150
 :last-transfer {:from :a :to :b :amount 30}}

State 3 (transfer-b-to-a 50):
{:balance-a 120
 :balance-b 30
 :total 150
 :last-transfer {:from :b :to :a :amount 50}}

INVARIANT VIOLATED: money-conserved
Expected: (+ balance-a balance-b) = total
Actual: (+ 120 30) = 150 ✓
```

Wait, that's not a violation! This is a false alarm. Let me show a real one:

**Real Counterexample** (with buggy concurrent transfer):
```
State 1 (Initial):
{:balance-a 100
 :balance-b 50
 :total 150
 :last-transfer nil}

State 2 (concurrent-transfer):
{:balance-a 70    ;; Two transfers read balance=100 simultaneously
 :balance-b 110   ;; Both subtract 30, but only one update visible
 :total 150       ;; VIOLATION: 70 + 110 = 180 ≠ 150
 :last-transfer {:from :a :to :b :amount 30}}
```

### Step-by-Step Debugging Process

<counterexample-analysis>
1. **Identify the violating state**: Which state breaks the invariant?
   - Look for "INVARIANT VIOLATED" marker
   - Note the state number

2. **Compute state diffs**: What changed between states?
   ```clojure
   ;; Helper function to diff states
   (defn state-diff [state1 state2]
     (into {}
       (for [k (keys state1)
             :when (not= (get state1 k)
                        (get state2 k))]
         [k {:old (get state1 k)
             :new (get state2 k)}])))
   ```

3. **Identify the action**: Which action caused the transition?
   - Read action name in trace
   - Review action's precondition and effect

4. **Question the model**:
   - Is the invariant too strict? (False positive)
   - Is the action missing a precondition? (Design flaw)
   - Did we model the environment correctly? (Incomplete model)

5. **Reproduce manually**: Can you construct this scenario by hand?
   - Draw state diagram
   - Walk through each transition
   - Understand WHY the violation occurs

6. **Fix the spec or design**:
   - Update action preconditions
   - Add missing state variables
   - Refine invariants
   - Re-run TLC

7. **Document the finding**:
   - Save counterexample trace
   - Write up the bug in plain English
   - Create ADR if design change needed
</counterexample-analysis>

### Saving Counterexample Traces

When a counterexample is found:

```bash
# Save trace
clj -X:test -m specs.my-spec-test 2>&1 | tee vault/specifications/formal/my-spec-trace.txt

# Create human-readable summary
cat > vault/specifications/formal/my-spec-summary.md <<EOF
# Specification: My System

## Date: $(date +%Y-%m-%d)

## Result: VIOLATION FOUND

### Invariant Violated
\`\`\`
money-conserved: Expected total = sum of balances
\`\`\`

### Counterexample Summary
1. Initial state: Both accounts have balance 100
2. Concurrent transfers: Two processes read same balance
3. Lost update: Only one write takes effect
4. Result: Money created out of thin air

### Root Cause
Missing concurrency control. Transfers need atomic read-modify-write.

### Fix
Add transaction ID and sequencing to ensure atomic updates.

### Next Steps
- [ ] Update spec with transaction model
- [ ] Re-verify with TLC
- [ ] Implement proper locking in code
- [ ] Create ADR for concurrency strategy
EOF
```

## Advanced Patterns

### Pattern 1: Non-Deterministic Choice

Model multiple possible values:

```clojure
;; In initial state
(def initial-state
  {::amount (r/one-of #{10 20 30})
   ::direction (r/one-of #{:a-to-b :b-to-a})})

;; In process logic
(r/defproc ^:fair transfer
  (fn [db]
    (let [success? (r/one-of #{true false})]  ;; Non-deterministic outcome
      (if success?
        (perform-transfer db)
        db))))  ;; Return unchanged on failure
```

### Pattern 2: Fairness (^:fair metadata)

Ensure actions eventually happen when enabled:

```clojure
;; Fair process: guaranteed to execute if enabled
(r/defproc ^:fair must-eventually-happen
  (fn [db]
    (when (enabled? db)
      (perform-action db))))

;; Without fairness, the model checker can ignore this action
(r/defproc may-never-happen
  (fn [db]
    (perform-action db)))
```

**Use `^:fair` for:**
- Actions that represent progress
- Liveness properties (they require fairness)
- Actions that must eventually happen

### Pattern 3: Temporal Properties

Specify liveness properties:

```clojure
;; Eventually reaches a state
(rh/defproperty eventual-completion
  [db]
  (rh/eventually (= (::status db) :completed)))

;; Repeatedly returns to a state (infinite recurrence)
(rh/defproperty always-eventually-idle
  [db]
  (rh/always (rh/eventually (= (::status db) :idle))))

;; Eventually stabilizes to a state (stabilization)
(rh/defproperty eventually-stable
  [db]
  (rh/eventually (rh/always (= (::status db) :completed))))
```

### Pattern 4: State Space Reduction with Constraints

Reduce state space by filtering out invalid states:

```clojure
(rh/defconstraint reasonable-bounds
  "Limit exploration to reasonable values"
  [db]
  (and (<= (::counter db) 100)
       (<= (count (::queue db)) 50)))

(rh/defconstraint no-negative-values
  "Filter out negative values early"
  [db]
  (and (>= (::balance-a db) 0)
       (>= (::balance-b db) 0)))
```

### Pattern 5: State Machine with Local PC

Use local program counter for multi-step processes:

```clojure
(r/defproc ^:fair multi-step-process
  {:local {:pc ::start}}  ;; Local state
  (fn [db local]
    (case (:pc local)

      ::start
      [db (r/goto ::step-2)]

      ::step-2
      [(update db ::counter inc)
       (r/goto ::step-3)]

      ::step-3
      [(assoc db ::status :done)
       (r/done)])))
```

## Integration with Development Workflow

### 1. Design Phase: Spec-First Development

<spec-first-workflow>
**Before writing code**:

1. **Read Gherkin scenarios**: What behaviors do we need?
   ```bash
   cat test/features/*.feature
   ```

2. **Create initial spec**: Model core algorithm
   ```bash
   vim src/specs/my_feature_spec.clj
   ```

3. **Run TLC**: Find design flaws early
   ```bash
   clj -X:test -m specs.my-feature-spec-test
   ```

4. **Iterate on design**: Fix spec, not code
   - Counterexample found? Update actions/invariants
   - Re-run TLC until verification passes

5. **Document verified design**:
   ```bash
   # Save successful verification
   echo "✓ Verified: $(date)" >> vault/specifications/formal/my-feature-spec.md
   ```

6. **Implement code**: Follow the verified spec
   - Spec is the blueprint
   - Code is the implementation
</spec-first-workflow>

### 2. Bug Investigation: Model the Bug

When you find a race condition or subtle bug:

1. **Reproduce in spec**: Model the exact scenario
2. **Add assertion**: Invariant that catches the bug
3. **Run TLC**: Should find violation
4. **Fix in spec**: Update actions to prevent bug
5. **Verify fix**: TLC should pass
6. **Apply to code**: Implement the fix from spec

### 3. Code Review: Verify Complex Logic

For complex concurrent logic:

1. **Request spec**: "Can you write a TLA+ spec for this?"
2. **Review spec**: Easier to reason about than code
3. **Run TLC**: Automated verification
4. **Approve with confidence**: Math proof > manual review

### 4. Refactoring: Prove Equivalence

Before refactoring complex logic:

1. **Spec old implementation**: Capture current behavior
2. **Spec new implementation**: Model refactored design
3. **Prove equivalence**: Both should satisfy same invariants
4. **Refactor safely**: Verified equivalent behavior

## Architecture Decision Records

When verification reveals design issues, document decisions:

<adr-template>
**File**: `vault/specifications/formal/adr-001-concurrency-control.md`

```markdown
# ADR 001: Use Pessimistic Locking for Account Transfers

## Status
Accepted

## Context
Formal verification with Recife revealed a lost update bug in concurrent account transfers:
- Two processes read the same balance simultaneously
- Both subtract transfer amounts
- Only one write persists
- Result: Money created from nothing (invariant violation)

See counterexample trace: account-transfer-trace.txt

## Decision
Implement pessimistic locking for account transfers:
1. Acquire lock on both accounts before reading
2. Perform transfer
3. Release locks

Verified in spec: `src/specs/account_transfer_with_locks_spec.clj`
TLC verified no invariant violations across 50,000 states.

## Consequences
Positive:
- Eliminates lost updates (proven by model checking)
- Simple to implement and reason about
- Consistent with other financial operations

Negative:
- Lower throughput (locks serialize transfers)
- Potential for deadlock (requires lock ordering)

## Alternatives Considered
1. Optimistic concurrency (version numbers)
   - More complex to model
   - Harder to reason about retry logic
2. Actor model (single writer per account)
   - Requires significant architecture change
   - May revisit in future

## Verification
- Spec: `src/specs/account_transfer_with_locks_spec.clj`
- Invariants checked:
  - Money conservation
  - No negative balances
  - Lock mutual exclusion
- States explored: 50,000
- Result: ✓ All invariants satisfied
```
</adr-template>

Suggest creating an ADR when:
- Counterexample reveals design flaw
- Verification influences architecture decision
- Trade-offs between correctness and performance

## Documentation Output

Always save specifications and results:

<documentation-structure>
```
vault/specifications/formal/
├── [feature-name]-spec.md          # Human-readable spec description
├── [feature-name]-trace.txt        # TLC counterexample (if violation found)
├── [feature-name]-results.txt      # TLC output (success or failure)
├── [feature-name]-notes.md         # Design notes, assumptions
└── adr-[NNN]-[decision].md         # Architecture decision record
```

**Content of spec.md**:
```markdown
# Feature Name Specification

## Purpose
[Why this spec exists]

## System Model
[What is being modeled]

## State Variables
- `variable-1`: [description]
- `variable-2`: [description]

## Actions
- `action-1`: [description, preconditions, effects]
- `action-2`: [description]

## Invariants
1. **[invariant-name]**: [English description]
   - TLA+: `[formal expression]`
   - Rationale: [why this must hold]

## Verification Results
- Date: [date]
- States explored: [number]
- Result: ✓ All invariants satisfied / ✗ Violation found
- Trace: [link to trace file if violation]

## Assumptions
- [assumption 1]
- [assumption 2]

## Future Work
- [ ] Model network partitions
- [ ] Add liveness properties
```
</documentation-structure>

## Quick API Reference

**Note:** For complete API documentation, see [`references/api.md`](references/api.md)

### Essential Functions

```clojure
;; Define process
(r/defproc ^:fair process-name
  (fn [db] ...))

;; Run model checker
@(r/run-model initial-state
              #{processes invariants properties}
              :trace-example true)

;; Define invariant
(rh/definvariant invariant-name
  [state]
  (boolean-expression state))

;; Define property
(rh/defproperty property-name
  [state]
  (rh/eventually condition))
```

### Key Concepts

- **`r/defproc`**: Define state transitions (actions)
- **`r/run-model`**: Execute model checking
- **`rh/definvariant`**: Safety properties (always hold)
- **`rh/defproperty`**: Liveness properties (eventually hold)
- **`r/one-of`**: Non-deterministic choice
- **`r/done`**: Mark process completion
- **`^:fair`**: Fairness annotation (required for liveness)

**See also:**
- [`references/api.md`](references/api.md) - Complete API reference
- [`references/examples.md`](references/examples.md) - Full working examples
- [`references/tlc-integration.md`](references/tlc-integration.md) - TLC integration details

## Learning Resources

After creating your first Recife spec:

1. **Understand TLA+ concepts**: Use `tla-concepts.md` skill for theory
2. **Study examples**: Look at Recife repository examples
3. **Read real specs**: AWS, Azure published specifications
4. **Practice**: Model systems from your codebase

## Troubleshooting

### Model Checking Takes Too Long

If model checking is slow:

1. **Reduce non-deterministic choices**:
   ```clojure
   ;; Instead of all possible values
   {::balance (r/one-of #{0 10 20 30 40 50 60 70 80 90 100})}

   ;; Use representative values
   {::balance (r/one-of #{0 50 100})}
   ```

2. **Add constraints to prune state space**:
   ```clojure
   (rh/defconstraint limit-exploration
     [db]
     (<= (::counter db) 50))
   ```

3. **Bound sequences**:
   ```clojure
   (defn bounded-conj [coll elem max-size]
     (if (< (count coll) max-size)
       (conj coll elem)
       coll))
   ```

4. **Simplify specification**: Test with minimal features first

### False Positives (Violations That Seem Wrong)

If TLC reports violations that seem incorrect:

1. **Review invariant**: Is it too strict?
   ```clojure
   (rh/definvariant maybe-too-strict
     [db]
     ;; Check if this really needs to hold in ALL states
     (= (::status db) :active))
   ```

2. **Check initial state**: Are all initial states valid?
   ```clojure
   ;; Verify initial state satisfies invariants
   (type-ok initial-state)
   (business-rule-ok initial-state)
   ```

3. **Trace through manually**: Walk through counterexample step by step
4. **Review guards**: Do your guards correctly prevent invalid transitions?

### No Issues Found (But You Expected Some)

If TLC completes successfully but you expected violations:

1. **Check states explored**:
   ```clojure
   (:states-explored result)
   ;; Too few? Your guards may be too restrictive
   ```

2. **Verify actions are enabled**: Add temporary invariant that should fail
   ```clojure
   (rh/definvariant should-fail-if-reachable
     [db]
     (not= (::status db) :error))  ;; Should find error state if reachable
   ```

3. **Check guards**: Are they preventing exploration of problem states?
   ```clojure
   ;; Guard may be too strict
   (when (and (condition-1 db)
              (condition-2 db)
              (condition-3 db))  ;; Too many conditions?
     ...)
   ```

4. **Expand initial state**: Try more diverse starting states
   ```clojure
   {::value (r/one-of #{0 1 2 3})}  ;; Instead of just #{0}
   ```

### Deadlock Detected

**Error:** "Deadlock detected"

**Causes:**
- No actions are enabled
- Processes are not marked complete with `r/done`

**Solutions:**

1. **Add completion marker**:
   ```clojure
   (r/defproc ^:fair complete
     (fn [db]
       (when (finished? db)
         (r/done))))
   ```

2. **Check guard conditions**: Are they too restrictive?
3. **Use `:no-deadlock true`** if terminal states are expected:
   ```clojure
   @(r/run-model initial-state components :no-deadlock true)
   ```

### Temporal Property Violations

**Error:** "Temporal property violation: ..."

**Common causes:**
- Missing `^:fair` metadata on actions
- Property is not achievable
- Infinite loop avoids desired state

**Solutions:**

1. **Add fairness**:
   ```clojure
   (r/defproc ^:fair action-that-makes-progress
     (fn [db] ...))
   ```

2. **Check property is achievable**: Can the system actually reach the desired state?
3. **Review action logic**: Does progress action actually advance the system?

For detailed debugging guidance, see [`references/tlc-integration.md`](references/tlc-integration.md)

## Next Steps

You're ready to:

1. ✓ Check if Recife is in `deps.edn`
2. ✓ Choose a concurrent/distributed component from your codebase
3. ✓ Read its Gherkin scenarios (if they exist)
4. ✓ Create initial spec using the template
5. ✓ Run TLC and iterate on design
6. ✓ Document results in `vault/specifications/formal/`

Remember:
- **Start small**: Model core algorithm, ignore performance optimizations
- **Safety first**: Invariants before liveness properties
- **Iterate**: Spec → Verify → Fix → Repeat
- **Document**: Counterexamples are gold, save them
- **Connect to code**: Spec should inform implementation

For deeper TLA+ understanding, refer back to the `tla-concepts.md` skill.
