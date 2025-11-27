---
description: Use when learning or explaining TLA+ formal specification concepts. Educational skill for state machines, temporal logic, invariants, and formal property verification.
---

# TLA+ Concepts: Formal Specification Fundamentals

## Core Philosophy

TLA+ (Temporal Logic of Actions) is a formal specification language for designing and reasoning about concurrent and distributed systems. Unlike testing which explores specific executions, formal verification exhaustively checks all possible behaviors.

### Why Formal Specification?

**Testing shows the presence of bugs, not their absence.** Formal verification provides mathematical proof that your system satisfies its requirements across ALL possible executions, including:

- Race conditions that occur once in a million runs
- Edge cases involving specific timing or ordering
- Subtle invariant violations in concurrent systems
- Liveness properties (things that eventually happen)

### When to Use Formal Methods

Use formal specification when:

1. **Concurrency is involved**: Race conditions, locks, distributed consensus
2. **Correctness is critical**: Financial transactions, safety systems, security protocols
3. **Design is complex**: Multiple components with intricate interactions
4. **Before implementation**: Catching design flaws early is 10x cheaper than fixing production bugs
5. **Learning from counterexamples**: Model checker finds concrete violation scenarios

Don't use for:
- Simple CRUD operations
- UI layout and styling
- Performance optimization (though you can model performance properties)

## Foundational Concepts

### 1. State Machines

Everything in TLA+ is modeled as a state machine:

**State**: A snapshot of all variables at a point in time
```
State = {
  balance: 100,
  locked: false,
  owner: "alice"
}
```

**Initial State**: Where the system starts
```
Init == balance = 0 /\ locked = false
```

**Actions**: State transitions (what can happen)
```
Deposit(amount) ==
  /\ locked = false          \* Precondition
  /\ balance' = balance + amount  \* Effect on next state
  /\ locked' = locked        \* Unchanged variables (frame condition)
```

**Behaviors**: Sequences of states connected by actions
```
[State0] --Action1--> [State1] --Action2--> [State2] --Action3--> ...
```

The model checker explores ALL possible behaviors.

### 2. Temporal Logic

TLA+ uses temporal logic to reason about properties over time:

**Always (□)**: Property holds in every state
```
□(balance >= 0)  \* Balance is always non-negative
```

**Eventually (◇)**: Property holds in some future state
```
◇(status = "completed")  \* Request eventually completes
```

**Leads to (~>)**: If P happens, Q eventually happens
```
(request_sent) ~> (response_received)  \* Every request gets a response
```

**Think of behaviors as infinite sequences**:
- "Always P" means P is true in every state of the sequence
- "Eventually P" means P becomes true at some point (and might become false again)
- "P leads to Q" means whenever P becomes true, Q eventually becomes true afterward

### 3. Safety vs Liveness

**Safety Properties** (bad things never happen):
- Invariants: Conditions that must hold in every reachable state
- Examples:
  - "Balance never goes negative"
  - "At most one process holds the lock"
  - "Messages are delivered in order"
- Violations are finite: Model checker finds a trace ending in violation

**Liveness Properties** (good things eventually happen):
- Progress: System doesn't get stuck
- Examples:
  - "Every request eventually gets a response"
  - "Deadlock never occurs indefinitely"
  - "Fair scheduling: every thread eventually runs"
- Violations are infinite: Model checker finds a trace with a cycle that never satisfies the property

**Design Priority**: Always specify safety properties first. They're easier to check and violations are easier to understand.

### 4. Invariants

An invariant is a boolean expression that must be true in all reachable states.

**Types of Invariants**:

1. **Type Invariants**: Variables have expected types/ranges
   ```
   TypeOK == /\ balance \in Nat
             /\ status \in {"pending", "active", "completed"}
   ```

2. **Structural Invariants**: Data structure consistency
   ```
   StructureOK == Len(queue) <= MAX_SIZE
   ```

3. **Business Logic Invariants**: Domain rules
   ```
   BusinessRuleOK == (status = "completed") => (balance = expected_balance)
   ```

4. **Mutual Exclusion**: At most one process in critical section
   ```
   MutexOK == \A i, j \in Processes:
              (i /= j) => ~(in_critical[i] /\ in_critical[j])
   ```

**Writing Good Invariants**:
- Start simple: Type invariants first
- Be specific: "balance >= 0" not "balance is valid"
- Think negative: "bad thing never happens" not "only good things happen"
- Decompose: Multiple small invariants better than one large one

### 5. Primed Variables (Next State)

In TLA+, `variable'` (pronounced "variable prime") refers to the value in the NEXT state:

```
Increment == count' = count + 1
```

This is fundamental to specifying actions:
- Unprimed variables: Current state
- Primed variables: Next state
- Actions define the relationship between current and next state

**Frame Conditions**: Variables not mentioned in an action can change arbitrarily. Always specify what stays the same:

```
\* BAD: Other variables might change unexpectedly
Increment == count' = count + 1

\* GOOD: Explicit about what doesn't change
Increment == /\ count' = count + 1
             /\ locked' = locked
             /\ owner' = owner
```

Or use UNCHANGED:
```
Increment == /\ count' = count + 1
             /\ UNCHANGED <<locked, owner>>
```

## Thinking in Formal Properties

### From Requirements to Formal Specs

**Requirement**: "A user can transfer money between accounts if they have sufficient balance"

**Informal decomposition**:
1. What are the states? (account balances, transfer status)
2. What can happen? (transfer action)
3. What are the preconditions? (sufficient balance)
4. What are the postconditions? (balances updated, amounts conserved)
5. What must always be true? (total money conserved, balances non-negative)

**Formal specification**:
```
\* State
VARIABLES balance_a, balance_b, total

\* Type invariant
TypeOK == /\ balance_a \in Nat
          /\ balance_b \in Nat
          /\ total \in Nat

\* Initial state
Init == /\ balance_a = 100
        /\ balance_b = 50
        /\ total = 150

\* Action
Transfer(amount) ==
  /\ amount > 0
  /\ amount <= balance_a          \* Precondition: sufficient balance
  /\ balance_a' = balance_a - amount    \* Effect
  /\ balance_b' = balance_b + amount
  /\ total' = total               \* Frame condition

\* Safety invariant: Money is conserved
MoneyConserved == balance_a + balance_b = total

\* Safety invariant: Balances never negative
NoNegativeBalance == balance_a >= 0 /\ balance_b >= 0
```

### Gherkin to TLA+ Translation Patterns

**Gherkin Given/When/Then** maps naturally to TLA+ concepts:

**Given** (precondition) → Initial state or action precondition
```gherkin
Given the account has balance 100
```
```tla
Init == balance = 100
```

**When** (action) → State transition
```gherkin
When the user transfers 30 to another account
```
```tla
Transfer(30) == /\ balance >= 30
                /\ balance' = balance - 30
                /\ other_balance' = other_balance + 30
```

**Then** (postcondition) → Invariant or temporal property
```gherkin
Then the balance should be 70
```
```tla
\* As postcondition in action:
Transfer(amount) == /\ balance' = balance - amount

\* As invariant (if this should always hold after transfers):
BalanceCorrect == (last_action = "transfer") =>
                  (balance = old_balance - transfer_amount)
```

**More Complex Example**:

```gherkin
Scenario: Mutual exclusion in critical section
  Given two processes want to enter critical section
  When both processes try to enter simultaneously
  Then at most one process should be in the critical section at any time
```

```tla
VARIABLES in_critical, wants_entry

Init == /\ in_critical = [p \in Processes |-> FALSE]
        /\ wants_entry = [p \in Processes |-> FALSE]

RequestEntry(p) ==
  /\ wants_entry' = [wants_entry EXCEPT ![p] = TRUE]
  /\ UNCHANGED in_critical

EnterCritical(p) ==
  /\ wants_entry[p]
  /\ ~(\E q \in Processes : (q /= p) /\ in_critical[q])  \* No one else inside
  /\ in_critical' = [in_critical EXCEPT ![p] = TRUE]
  /\ UNCHANGED wants_entry

\* The "Then" becomes an invariant
MutualExclusion ==
  \A p, q \in Processes :
    (p /= q) => ~(in_critical[p] /\ in_critical[q])
```

## Common Patterns

### Pattern 1: Mutex (Mutual Exclusion)

**Problem**: Ensure only one process accesses shared resource

**Key Concepts**:
- State: Which processes are in critical section
- Safety: At most one process inside
- Liveness: Every process eventually enters (if it wants to)

**Skeleton**:
```
VARIABLES in_critical  \* in_critical[p] = is process p in critical section?

MutexSafety == \A p, q \in Processes:
               (p /= q) => ~(in_critical[p] /\ in_critical[q])

MutexLiveness == \A p \in Processes:
                 (wants_entry[p]) ~> (in_critical[p])
```

### Pattern 2: Eventual Consistency

**Problem**: Replicas eventually converge to same state

**Key Concepts**:
- State: Value at each replica, pending messages
- Safety: Convergence properties (e.g., no conflicts)
- Liveness: Eventually all replicas have same value

**Skeleton**:
```
VARIABLES replica_value, messages

EventuallyConsistent ==
  []<>(messages = {} =>
       \A r1, r2 \in Replicas: replica_value[r1] = replica_value[r2])
```

Read as: "Always eventually, if there are no pending messages, all replicas have the same value"

### Pattern 3: Request-Response

**Problem**: Every request gets a response

**Key Concepts**:
- State: Pending requests, sent responses
- Safety: Responses match requests
- Liveness: Every request eventually gets response

**Skeleton**:
```
VARIABLES requests, responses

RequestResponseMatch ==
  \A req \in requests:
    req \in responses => response_data[req] = expected_response(req)

EventualResponse ==
  \A req \in requests: <>(req \in responses)
```

### Pattern 4: State Machine Replication

**Problem**: Multiple replicas execute same operations in same order

**Key Concepts**:
- State: Operation log at each replica
- Safety: All replicas have same log prefix
- Liveness: Operations eventually appear in all logs

**Skeleton**:
```
VARIABLES log  \* log[replica] = sequence of operations

LogConsistency ==
  \A r1, r2 \in Replicas:
    \A i \in 1..Min(Len(log[r1]), Len(log[r2])):
      log[r1][i] = log[r2][i]  \* Same operation at same index
```

## Educational Resources

### Leslie Lamport's Materials

- **Video Course**: "TLA+ Video Course" by Leslie Lamport
  - https://lamport.azurewebsites.net/video/videos.html
  - Start here for foundational understanding

- **Specifying Systems**: The canonical TLA+ book
  - https://lamport.azurewebsites.net/tla/book.html
  - Free PDF, comprehensive reference

- **Hyperbook**: Interactive TLA+ learning
  - https://lamport.azurewebsites.net/tla/hyperbook.html

### Practical Learning Path

1. **Week 1**: Watch Lamport's video lectures (1-3)
   - Understand state machines, actions, invariants

2. **Week 2**: Write first spec (simple counter, mutex)
   - Focus on safety properties
   - Learn to read TLC output

3. **Week 3**: Add temporal properties
   - Liveness: eventual consistency, fairness
   - Understand counterexample traces

4. **Week 4**: Real system (your distributed cache, message queue)
   - Start small: model core algorithm only
   - Incrementally add complexity

### Community Examples

- **TLA+ Examples Repository**: https://github.com/tlaplus/Examples
- **Real-world specs**: Amazon (S3, DynamoDB), Microsoft (Azure), MongoDB
- **Dr. TLA+ Series**: Practical patterns blog by Hillel Wayne

## Glossary

**Action**: A state transition, relating current state to next state. Written with primed variables.

**Behavior**: An infinite sequence of states where each transition is allowed by the spec.

**Fairness**: Constraint that prevents infinite stuttering. Ensures actions eventually happen.

**Invariant**: A boolean expression true in all reachable states. Safety property.

**Liveness**: Property that good things eventually happen. Uses ◇ (eventually).

**Model Checking**: Automated verification that exhaustively explores state space.

**Primed Variable**: `var'` refers to value in next state. Used in actions.

**Safety**: Property that bad things never happen. Uses □ (always).

**State**: Complete assignment of values to all variables at a point in time.

**State Space**: Set of all possible states the system can reach.

**Stuttering**: Action that doesn't change state. TLA+ allows infinite stuttering unless fairness is specified.

**Temporal Logic**: Logic for reasoning about sequences of states over time.

**TLC**: The TLA+ model checker. Explores state space to find invariant violations.

**Trace**: A sequence of states showing a specific execution path. Counterexamples are traces.

## Pedagogical Approach

When teaching TLA+:

1. **Start with concrete examples**: Real bugs, race conditions
2. **Draw state diagrams**: Visual intuition before formal syntax
3. **Show counterexamples**: Model checker output is the best teacher
4. **Iterate**: Spec, check, fix, repeat
5. **Connect to code**: "This TLA+ action is like this function"
6. **Emphasize thinking**: TLA+ is more about clear thinking than syntax

**Common Beginner Mistakes**:

- Forgetting frame conditions (UNCHANGED)
- Confusing safety and liveness
- Over-specifying implementation details
- Not starting with type invariants
- Making state space too large

**Debugging Mindset**:

When TLC finds a violation:
1. **Read the trace**: What sequence of states led to the violation?
2. **Identify the bad state**: Which state violates the invariant?
3. **Work backwards**: What action created that state?
4. **Question assumptions**: Is your invariant too strong? Action too weak?
5. **Reproduce**: Can you construct this scenario by hand?

## Integration with Development

**When to Write Specs**:

- **Design phase**: Before writing code, spec the algorithm
- **Bug investigation**: Found a race condition? Spec it to understand
- **Code review**: Complex concurrent logic? Spec to verify
- **Refactoring**: Prove new implementation equivalent to old

**Workflow**:

1. Write Gherkin scenarios (human-readable requirements)
2. Translate to TLA+ spec (formal model)
3. Run TLC to find design flaws
4. Fix spec, re-verify
5. Implement code following verified spec
6. Keep spec as living documentation

**Documentation Output**:

Store specifications in `vault/specifications/formal/`:
```
vault/specifications/formal/
├── account-transfer.tla
├── account-transfer-trace.txt  (counterexample if found)
└── account-transfer-notes.md   (design decisions)
```

## Next Steps

After understanding these concepts:

1. **Try recife-modeling.md skill**: Practical Clojure-based TLA+ modeling
2. **Model a real system**: Pick something with concurrency from your codebase
3. **Join TLA+ community**: Google group, Slack, conferences
4. **Read real specs**: AWS, Azure published specs show real-world complexity

Remember: **The goal is not perfect specs, but clearer thinking about system behavior.**
