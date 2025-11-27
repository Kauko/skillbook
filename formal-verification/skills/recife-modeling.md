---
description: Use when creating formal specifications in Clojure using Recife. Translates Gherkin scenarios to TLA+ predicates, runs TLC model checker, and interprets results.
---

# Recife Modeling: TLA+ in Clojure

## Overview

Recife is a Clojure library that brings TLA+ model checking to Clojure. Instead of learning TLA+'s syntax, write specifications as Clojure data structures and functions.

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
  (:require [recife.core :as r]))

;; ====================
;; State Space
;; ====================

(def initial-state
  "Defines all possible initial states"
  {:variable-1 0
   :variable-2 #{"possible" "values"}
   :variable-3 []})

;; ====================
;; Actions
;; ====================

(defn action-1
  "Description of what this action does

   Preconditions:
   - [CONDITION 1]
   - [CONDITION 2]

   Effects:
   - [CHANGE 1]
   - [CHANGE 2]"
  [state params]
  (when (precondition? state params)  ;; Guard
    (-> state
        (update :variable-1 inc)
        (update :variable-2 conj (:new-value params)))))

(defn action-2
  "Description of what this action does"
  [state params]
  (when (another-precondition? state)
    (assoc state :variable-3 (compute-next-value state))))

;; ====================
;; Specification
;; ====================

(def spec
  "The complete system specification"
  (r/spec
    {:init initial-state
     :next [action-1 action-2]  ;; Non-deterministic choice
     :invariants [type-ok business-rule-ok]
     :temporal-properties [liveness-property]}))

;; ====================
;; Invariants
;; ====================

(defn type-ok
  "Type invariant: variables have expected types/ranges"
  [state]
  (and (nat-int? (:variable-1 state))
       (set? (:variable-2 state))
       (vector? (:variable-3 state))))

(defn business-rule-ok
  "Business logic invariant: [DESCRIBE RULE]"
  [state]
  (implies (:condition state)
           (:expected-property state)))

;; ====================
;; Temporal Properties
;; ====================

(defn liveness-property
  "Eventually property: [DESCRIBE WHAT EVENTUALLY HAPPENS]"
  [state]
  (r/eventually
    (= (:status state) :completed)))

;; ====================
;; Model Configuration
;; ====================

(def model-config
  "Configuration for TLC model checker"
  {:max-states 100000       ;; State space limit
   :max-trace-length 50     ;; Max steps in counterexample
   :workers 4               ;; Parallel workers
   :symmetry-sets []})      ;; Symmetry reduction

;; ====================
;; Helpers
;; ====================

(defn implies [p q]
  "Logical implication: p => q"
  (or (not p) q))

(defn precondition? [state params]
  "Check if action preconditions are satisfied"
  (and (:some-flag state)
       (>= (:variable-1 state) (:threshold params))))
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
  (:require [recife.core :as r]))

;; ====================
;; State Space
;; ====================

(def initial-state
  "Initial state space: various starting balances"
  {:balance-a #{0 20 50 100}      ;; Non-deterministic initial values
   :balance-b #{0 50 100}
   :total nil                      ;; Computed from balances
   :last-transfer nil})            ;; Track last operation

(defn compute-total [state]
  (+ (:balance-a state) (:balance-b state)))

(defn init [state]
  "Initialize with consistent total"
  (assoc state :total (compute-total state)))

;; ====================
;; Actions
;; ====================

(defn transfer-a-to-b
  "Transfer amount from account A to B

   Preconditions:
   - amount > 0
   - account A has sufficient balance

   Effects:
   - Decrease balance-a by amount
   - Increase balance-b by amount
   - Total remains unchanged (conservation)"
  [state amount]
  (when (and (pos? amount)
             (>= (:balance-a state) amount))  ;; Guard: sufficient balance
    (-> state
        (update :balance-a - amount)
        (update :balance-b + amount)
        (assoc :last-transfer {:from :a :to :b :amount amount}))))

(defn transfer-b-to-a
  "Transfer amount from account B to A"
  [state amount]
  (when (and (pos? amount)
             (>= (:balance-b state) amount))
    (-> state
        (update :balance-b - amount)
        (update :balance-a + amount)
        (assoc :last-transfer {:from :b :to :a :amount amount}))))

;; Generate multiple transfer amounts to explore
(defn all-transfer-actions [state]
  (for [amount [10 20 30 50]
        action [transfer-a-to-b transfer-b-to-a]]
    (action state amount)))

;; ====================
;; Invariants
;; ====================

(defn type-ok
  "Type invariant: balances are natural numbers"
  [state]
  (and (nat-int? (:balance-a state))
       (nat-int? (:balance-b state))
       (nat-int? (:total state))))

(defn money-conserved
  "Safety invariant: total money is always conserved"
  [state]
  (= (+ (:balance-a state) (:balance-b state))
     (:total state)))

(defn no-negative-balance
  "Safety invariant: balances never go negative"
  [state]
  (and (>= (:balance-a state) 0)
       (>= (:balance-b state) 0)))

;; ====================
;; Specification
;; ====================

(def spec
  (r/spec
    {:init (map init initial-state)
     :next all-transfer-actions
     :invariants [type-ok money-conserved no-negative-balance]}))

;; ====================
;; Model Configuration
;; ====================

(def model-config
  {:max-states 50000
   :max-trace-length 20
   :workers 4})
```

**Test File** (`test/specs/account_transfer_spec_test.clj`):

```clojure
(ns specs.account-transfer-spec-test
  (:require [clojure.test :refer :all]
            [recife.core :as r]
            [specs.account-transfer-spec :as spec]))

(deftest account-transfer-verification
  (testing "Account transfer satisfies all invariants"
    (let [result (r/check spec/spec spec/model-config)]
      (is (:success? result)
          (str "Invariant violation found:\n"
               (r/format-counterexample (:counterexample result)))))))
```

### Running the Model Checker

```bash
# Run verification
clj -X:test -m specs.account-transfer-spec-test

# Or with verbose output
TLC_DEBUG=true clj -X:test -m specs.account-transfer-spec-test

# Save results
clj -X:test -m specs.account-transfer-spec-test | tee vault/specifications/formal/account-transfer-results.txt
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

Model multiple possible actions:

```clojure
(defn next-actions [state]
  "Non-deterministic choice among multiple actions"
  (concat
    (for [amount [10 20 30]]
      (transfer-a-to-b state amount))
    (for [amount [5 15]]
      (transfer-b-to-a state amount))
    [(stutter state)]))  ;; Include stuttering (no-op)

(defn stutter [state]
  "Action that doesn't change state (allowed in TLA+)"
  state)
```

### Pattern 2: Fairness Constraints

Ensure actions eventually happen:

```clojure
(def spec-with-fairness
  (r/spec
    {:init initial-state
     :next all-actions
     :invariants [safety-invariant]
     :fairness [:weak-fair transfer-a-to-b  ;; Eventually happens if enabled
                :strong-fair transfer-b-to-a]}))  ;; Infinitely often enabled => infinitely often happens
```

### Pattern 3: Temporal Properties

Specify liveness properties:

```clojure
(defn eventual-completion
  "Every request eventually completes"
  [state]
  (r/leads-to
    (contains? (:pending-requests state) :req-123)  ;; If request exists
    (contains? (:completed-requests state) :req-123)))  ;; Eventually completed

(defn always-eventually-idle
  "System repeatedly returns to idle state"
  [state]
  (r/always (r/eventually (= (:status state) :idle))))
```

### Pattern 4: Symmetry Reduction

Reduce state space with symmetry:

```clojure
(def model-config-with-symmetry
  {:max-states 100000
   :symmetry-sets [[:process-1 :process-2 :process-3]  ;; Processes are symmetric
                   [:account-a :account-b]]})           ;; Accounts are symmetric
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

## Reference: Recife API

### Core Functions

```clojure
;; Create specification
(r/spec {:init initial-states
         :next action-functions
         :invariants invariant-predicates
         :temporal-properties temporal-properties
         :fairness fairness-constraints})

;; Run model checker
(r/check spec config)
;; Returns: {:success? boolean
;;           :states-explored number
;;           :counterexample trace-if-violation}

;; Temporal operators
(r/always pred)           ;; □pred - always holds
(r/eventually pred)       ;; ◇pred - eventually holds
(r/leads-to p q)         ;; p ~> q - p implies eventually q
(r/unless p q)           ;; p unless q

;; Format counterexample for humans
(r/format-counterexample trace)
```

### Helper Patterns

```clojure
;; Logical implication
(defn implies [p q]
  (or (not p) q))

;; Universal quantification
(every? predicate collection)

;; Existential quantification
(some predicate collection)

;; Set operations
(clojure.set/subset? s1 s2)
(clojure.set/intersection s1 s2)
(clojure.set/difference s1 s2)
```

## Learning Resources

After creating your first Recife spec:

1. **Understand TLA+ concepts**: Use `tla-concepts.md` skill for theory
2. **Study examples**: Look at Recife repository examples
3. **Read real specs**: AWS, Azure published specifications
4. **Practice**: Model systems from your codebase

## Troubleshooting

### TLC Takes Too Long

If model checking is slow:

1. **Reduce state space**:
   ```clojure
   ;; Instead of all possible values
   {:balance #{0 10 20 30 40 50 60 70 80 90 100}}

   ;; Use representative values
   {:balance #{0 50 100}}
   ```

2. **Add symmetry**:
   ```clojure
   {:symmetry-sets [[:account-a :account-b]]}
   ```

3. **Bound sequences**:
   ```clojure
   (defn bounded-append [vec elem]
     (if (< (count vec) MAX-LENGTH)
       (conj vec elem)
       vec))
   ```

4. **Check fewer states**:
   ```clojure
   {:max-states 10000}  ;; Start small, increase if needed
   ```

### False Positives

If TLC reports violations that seem wrong:

1. **Review invariant**: Is it too strict?
2. **Check initial state**: Are all initial states valid?
3. **Trace through manually**: Draw state diagram
4. **Add debug logging**: Print state in action functions

### TLC Finds Nothing

If TLC completes but finds no issues (and you expected some):

1. **Check state space**: Is it being explored?
   ```clojure
   (println "States explored:" (:states-explored result))
   ```

2. **Verify actions are reachable**: Add temporary invariant that should fail
   ```clojure
   (defn should-fail [state]
     (not= (:status state) :error))  ;; Should find error state
   ```

3. **Increase max-states**: May not explore deep enough

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
