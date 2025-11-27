# Invariants and Properties in TLA+

## Overview

Properties describe what your system should (or should not) do. TLA+ distinguishes between **safety properties** (bad things never happen) and **liveness properties** (good things eventually happen).

## Safety Properties

**Definition:** Bad things don't happen.

Safety properties are about **preventing** undesired states or sequences. They can be violated in finite time - once the bad thing happens, the violation is permanent.

### Characteristics

- **Finite counterexamples:** A safety violation is a finite trace ending in a bad state
- **Checkable per-state:** Usually (but not always) can be verified by examining individual states
- **Fast to verify:** TLC checks safety properties efficiently during state space exploration

### Examples

**State-based safety:**
- Two threads cannot both be in a critical section simultaneously
- User IDs must be unique
- RAM usage never exceeds 500 KB
- Balance never goes negative

**Sequence-based safety:**
- Traffic lights cannot go directly from green to red
- Messages are delivered at most once
- Operations execute in causal order

## Invariants

An **invariant** is a boolean formula that must be true in all reachable states. Invariants are the most common type of safety property.

### Type Invariants

Define the valid types and ranges for variables:

```tla
TypeOK ==
  /\ balance \in Nat                          \* Natural number
  /\ status \in {"pending", "active", "done"} \* Enumerated type
  /\ queue \in Seq(Messages)                  \* Sequence of messages
  /\ locks \in [Processes -> BOOLEAN]         \* Function from processes to booleans
```

**Always write type invariants first.** They catch basic modeling errors and document your data structures.

### Structural Invariants

Express constraints on data structure properties:

```tla
StructureOK ==
  /\ Len(queue) <= MAX_QUEUE_SIZE
  /\ Cardinality(active_users) <= MAX_CONNECTIONS
  /\ \A p \in Processes: Len(message_buffer[p]) <= BUFFER_SIZE
```

### Business Logic Invariants

Capture domain-specific correctness requirements:

```tla
BusinessInvariant ==
  /\ total_debits = total_credits  \* Accounting invariant
  /\ \A account \in Accounts:
       (status[account] = "closed") => (balance[account] = 0)
  /\ \A txn \in Transactions:
       (txn.state = "committed") => (txn \in ledger)
```

### Mutual Exclusion Invariants

Ensure exclusive access to shared resources:

```tla
MutualExclusion ==
  \A p1, p2 \in Processes:
    (p1 /= p2) => ~(in_critical[p1] /\ in_critical[p2])
```

Read as: "For any two distinct processes, they cannot both be in the critical section."

### Consistency Invariants

Express relationships between variables:

```tla
ConsistencyInvariant ==
  /\ pending_requests \subseteq all_requests
  /\ completed_requests \cap pending_requests = {}
  /\ completed_requests \cup pending_requests \subseteq all_requests
```

## Writing Effective Invariants

### 1. Start Simple

Begin with type invariants, then add complexity:

```tla
\* Stage 1: Types only
TypeOK ==
  /\ balance \in Nat
  /\ locked \in BOOLEAN

\* Stage 2: Add simple constraints
SimpleInvariant ==
  /\ TypeOK
  /\ balance <= MAX_BALANCE

\* Stage 3: Add complex business logic
FullInvariant ==
  /\ SimpleInvariant
  /\ (locked = TRUE) => (balance >= MIN_LOCKED_BALANCE)
```

### 2. Be Specific

```tla
\* Too vague
BalanceValid == balance >= 0

\* Better: Explicit bounds
BalanceValid ==
  /\ balance >= 0
  /\ balance <= MAX_BALANCE
  /\ balance % 1 = 0  \* Must be whole number (if using Reals)
```

### 3. Think Negative

Frame invariants as "bad thing never happens" rather than "only good things happen":

```tla
\* Good: States what should NOT happen
NoDataLoss ==
  /\ \A msg \in sent_messages:
       (msg \in delivered_messages) \/ (msg \in in_flight_messages)

\* Less clear: States what SHOULD be true
DataIntegrity == delivered_messages \subseteq sent_messages
```

### 4. Decompose Complex Invariants

Multiple small invariants are better than one large one:

```tla
\* Hard to debug when it fails
ComplexInvariant ==
  /\ balance >= 0
  /\ locked \in BOOLEAN
  /\ (locked = TRUE) => (owner /= NULL)
  /\ Len(history) <= MAX_HISTORY

\* Better: Separate concerns
BalanceInvariant == balance >= 0
TypeInvariant == locked \in BOOLEAN
LockInvariant == (locked = TRUE) => (owner /= NULL)
HistoryInvariant == Len(history) <= MAX_HISTORY
```

When an invariant fails, TLC tells you which one - making debugging much easier.

## Liveness Properties

**Definition:** Good things eventually happen.

Liveness properties assert that the system makes progress toward desired goals. They cannot be violated in finite time - only infinite behaviors can show liveness violations.

### Characteristics

- **Infinite counterexamples:** Liveness violations are infinite traces (usually with a cycle)
- **Not checkable per-state:** Must examine entire behaviors
- **Slow to verify:** TLC must explore behavior cycles, much slower than safety checks

### Examples

**Eventual outcomes:**
- Every request eventually receives a response
- Database eventually becomes consistent
- All queued tasks eventually complete
- Deadlock never occurs indefinitely

**Fair scheduling:**
- Every thread eventually gets CPU time
- Every process eventually makes progress
- Messages eventually arrive

## Distinguishing Safety from Liveness

**Key test:** Can the property be violated in finite time?

- **If yes → Safety:** Once violated, it stays violated
- **If no → Liveness:** Only infinite behaviors show violations

### Example: Message Delivery

```tla
\* SAFETY: Message delivered at most once
AtMostOnce ==
  []\A msg \in Messages:
    Cardinality({i \in 1..Len(delivered) : delivered[i] = msg}) <= 1

\* LIVENESS: Message eventually delivered
EventualDelivery ==
  \A msg \in Messages:
    (msg \in sent) ~> (msg \in delivered)
```

`AtMostOnce` can be violated at a specific moment (message delivered twice). `EventualDelivery` requires checking the entire infinite behavior.

## Common Property Patterns

### Pattern 1: Request-Response

```tla
\* Safety: Responses match requests
ResponseMatches ==
  []\A resp \in responses:
    \E req \in requests:
      /\ resp.id = req.id
      /\ resp.data = ExpectedResponse(req)

\* Liveness: Every request gets a response
GuaranteedResponse ==
  \A req \in requests:
    (req \in sent) ~> (req \in acknowledged)
```

### Pattern 2: Resource Management

```tla
\* Safety: Resource not over-allocated
NoOverAllocation ==
  []allocated_memory <= total_memory

\* Liveness: Released resources eventually reusable
ResourceRecycling ==
  \A res \in Resources:
    (res \in released) ~> (res \in available)
```

### Pattern 3: State Machine Progress

```tla
\* Safety: States follow valid transitions
ValidTransitions ==
  []\A state \in States:
    (state' /= state) => ValidTransition(state, state')

\* Liveness: Eventually reach terminal state
EventualTermination ==
  <>[]state \in TerminalStates
```

### Pattern 4: Consensus

```tla
\* Safety: Agreement - no two processes decide differently
Agreement ==
  []\A p1, p2 \in Processes:
    (decided[p1] /= NULL /\ decided[p2] /= NULL)
    => (decided[p1] = decided[p2])

\* Safety: Validity - decided value was proposed
Validity ==
  []\A p \in Processes:
    (decided[p] /= NULL) => (decided[p] \in proposed_values)

\* Liveness: Termination - all processes eventually decide
Termination ==
  \A p \in Processes: <>(decided[p] /= NULL)
```

### Pattern 5: Eventual Consistency

```tla
\* Safety: Replicas don't conflict
NoConflicts ==
  []\A r1, r2 \in Replicas:
    Compatible(state[r1], state[r2])

\* Liveness: Replicas eventually converge
EventualConsistency ==
  <>[](no_pending_updates =>
       \A r1, r2 \in Replicas: state[r1] = state[r2])
```

## Inductive Invariants

An **inductive invariant** is an invariant that:
1. Holds in all initial states
2. If it holds in state S, and action A is enabled, it holds in the next state S'

Inductive invariants are easier to prove formally (using TLAPS) because you only need to check:
- `Init => Inv`
- `Inv /\ Next => Inv'`

Rather than exploring the entire state space.

### Example

```tla
\* This is inductive
MoneyConserved ==
  balance_a + balance_b = total

\* Proof obligations:
\* 1. Init => MoneyConserved
\*    100 + 50 = 150 ✓
\*
\* 2. MoneyConserved /\ Transfer => MoneyConserved'
\*    (balance_a + balance_b = total) /\
\*    (balance_a' = balance_a - amt /\ balance_b' = balance_b + amt /\ total' = total)
\*    => (balance_a' + balance_b' = total')
\*    => ((balance_a - amt) + (balance_b + amt) = total)
\*    => (balance_a + balance_b = total) ✓
```

## Checking Properties in TLC

### Safety Properties (Invariants)

In TLC model, add to "Invariants" section:
```
TypeOK
BalanceNonNegative
MutualExclusion
```

TLC checks these **in every state** during exploration.

**Performance:** Very fast. Checked during normal state space exploration.

### Liveness Properties

In TLC model, add to "Properties" section:
```
Spec
EventualTermination
[]<>FairScheduled
```

TLC checks these **over entire behaviors**, looking for cycles that violate the property.

**Performance:** Much slower. Requires cycle detection and behavior analysis.

**Best practice:**
- Use large models for safety checking
- Use smaller models for liveness checking

## Debugging Failed Properties

### Safety Violation

TLC provides a **finite trace** ending in the bad state:

```
Error: Invariant BalanceNonNegative is violated
State 1: balance = 100, locked = false
State 2: balance = 50, locked = false  (Withdraw(50))
State 3: balance = -20, locked = false (Withdraw(70))  <-- VIOLATION
```

**Debug approach:**
1. Look at the final state - what's wrong?
2. Work backward - which action caused it?
3. Check preconditions - why was that action enabled?
4. Fix the action or strengthen preconditions

### Liveness Violation

TLC provides a **trace with a cycle** showing progress never happens:

```
Error: Temporal property violated
State 1: request_sent = true, response = null
State 2: processing = true, response = null
State 3: processing = true, response = null
[Back to State 2] <-- Cycle: response never arrives
```

**Debug approach:**
1. Identify the cycle - what states loop forever?
2. Check fairness - is the needed action marked as fair?
3. Check enablement - is the action actually enabled in the cycle?
4. Add fairness or strengthen liveness properties

## Common Mistakes

### Mistake 1: Mixing Safety and Liveness

```tla
\* Wrong: This is liveness, not safety
SafetyProperty == <>(status = "completed")

\* Right: Safety is about current state
SafetyProperty == (status = "failed") => (error_logged = true)
```

### Mistake 2: Invariants That Are Too Strong

```tla
\* Too strong: Forbids all interleavings
TooStrong == (processing_a = true) => (processing_b = false)

\* Right: Only forbid problematic states
JustRight == ~(in_critical_a /\ in_critical_b)
```

### Mistake 3: Forgetting Fairness for Liveness

```tla
\* This will fail without fairness
EventualCompletion == <>(all_tasks_done)

\* Add to Spec:
Spec == Init /\ [][Next]_vars /\ WF_vars(ProcessTask)
```

### Mistake 4: Not Testing Type Invariants

Always check `TypeOK` first. Many bugs hide as type violations:

```tla
\* Model finds: balance = -5
\* But you specified: balance \in Nat

TypeOK == balance \in Nat  \* Catches the bug immediately
```

## Further Reading

- Hillel Wayne's safety vs liveness: [Safety and Liveness Properties](https://www.hillelwayne.com/post/safety-and-liveness/)
- Jack Vanlightly on liveness: [The Importance of Liveness Properties](https://jack-vanlightly.com/blog/2023/10/10/the-importance-of-liveness-properties-part1)
- Learn TLA+ properties: [Temporal Properties](https://learntla.com/core/temporal-logic.html)
- Verifying safety with TLAPS: [Verifying Safety Properties](https://link.springer.com/chapter/10.1007/978-3-642-14203-1_12)
- Wikipedia on Linear Temporal Logic: [Linear Temporal Logic](https://en.wikipedia.org/wiki/Linear_temporal_logic)
