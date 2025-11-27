---
description: Use when learning or explaining TLA+ formal specification concepts. Educational skill for state machines, temporal logic, invariants, and formal property verification.
---

# TLA+ Concepts: Formal Specification Fundamentals

## What is TLA+?

TLA+ (Temporal Logic of Actions) is a formal specification language for designing and reasoning about concurrent and distributed systems. Unlike testing which explores specific executions, formal verification exhaustively checks all possible behaviors.

**Core insight:** Testing shows the presence of bugs, not their absence. TLA+ provides mathematical proof that your system satisfies its requirements across ALL possible executions.

## When to Use TLA+

**Use formal specification when:**

1. **Concurrency is involved** - Race conditions, locks, distributed consensus
2. **Correctness is critical** - Financial transactions, safety systems, security protocols
3. **Design is complex** - Multiple components with intricate interactions
4. **Before implementation** - Catching design flaws early is 10x cheaper than fixing production bugs
5. **Learning from counterexamples** - Model checker finds concrete violation scenarios

**Don't use for:**
- Simple CRUD operations
- UI layout and styling
- Performance optimization (though you can model performance properties)

## Key Concepts

### State Machines

Everything in TLA+ is a state machine. A behavior is an infinite sequence of states:

```
[State0] --Action1--> [State1] --Action2--> [State2] --Action3--> ...
```

**Core components:**
- **State:** Complete snapshot of all variables
- **Init:** Predicate defining valid starting states
- **Next:** Disjunction of all possible state transitions
- **Actions:** Formulas relating current state (unprimed) to next state (primed)

```tla
VARIABLES balance, locked

Init == balance = 0 /\ locked = FALSE

Deposit ==
  /\ locked = FALSE
  /\ balance' = balance + 100
  /\ UNCHANGED locked

Spec == Init /\ [][Next]_vars
```

**Critical rule:** Always specify frame conditions - what stays the same.

**Deep dive:** See [references/state-machines.md](references/state-machines.md) for Init, Next, actions, primed variables, stuttering, and state space exploration.

### Temporal Logic

Temporal operators express properties over time:

**□ (Always):** Property holds in every state
```tla
[](balance >= 0)  \* Balance always non-negative
```

**◇ (Eventually):** Property holds in some future state
```tla
<>(status = "completed")  \* Eventually completes
```

**~> (Leads-to):** If P happens, Q eventually happens
```tla
(request_sent) ~> (response_received)  \* Every request gets response
```

**Combined operators:**
- **◇□P** - Eventually always P (permanent convergence)
- **□◇P** - Always eventually P (infinitely often)

**Fairness constraints** prevent infinite stuttering:
- **WF_vars(Action)** - Weak fairness: if continuously enabled, must occur
- **SF_vars(Action)** - Strong fairness: if repeatedly enabled, must occur

**Deep dive:** See [references/temporal-logic.md](references/temporal-logic.md) for detailed operator semantics, fairness, and debugging liveness properties.

### Safety vs Liveness

**Safety Properties** (bad things never happen):
- Violations are **finite** - single trace to bad state
- Examples: mutex, no negative balance, messages delivered at most once
- Fast to verify with TLC

**Liveness Properties** (good things eventually happen):
- Violations are **infinite** - cycle showing progress never happens
- Examples: eventual termination, every request gets response, deadlock freedom
- Slow to verify - requires cycle detection

**Design priority:** Always specify safety properties first.

**Deep dive:** See [references/invariants-properties.md](references/invariants-properties.md) for writing invariants, distinguishing safety/liveness, and debugging violations.

### Invariants

Boolean formulas true in all reachable states:

```tla
\* Type invariant
TypeOK ==
  /\ balance \in Nat
  /\ status \in {"pending", "active", "completed"}

\* Business invariant
MoneyConserved == balance_a + balance_b = total

\* Mutual exclusion
MutexOK ==
  \A i, j \in Processes:
    (i /= j) => ~(in_critical[i] /\ in_critical[j])
```

**Best practices:**
- Start with type invariants
- Decompose into multiple small invariants
- Think negative: "bad thing never happens"
- Be specific about bounds and constraints

**Deep dive:** See [references/invariants-properties.md](references/invariants-properties.md) for types of invariants, inductive invariants, and checking properties in TLC.

## From Requirements to Formal Specs

**Gherkin to TLA+ translation:**

| Gherkin | TLA+ |
|---------|------|
| **Given** (precondition) | Initial state or action precondition |
| **When** (action) | State transition |
| **Then** (postcondition) | Invariant or temporal property |

**Example:**
```gherkin
Scenario: Account transfer
  Given account A has balance 100
  When user transfers 30 to account B
  Then account A balance is 70
  And total money is conserved
```

```tla
Init == balance_a = 100 /\ balance_b = 0

Transfer(amount) ==
  /\ amount <= balance_a
  /\ balance_a' = balance_a - amount
  /\ balance_b' = balance_b + amount

MoneyConserved == balance_a + balance_b = 100
```

## Common Patterns

### Pattern 1: Mutual Exclusion
**Safety:** At most one process in critical section
```tla
MutexSafety ==
  \A p, q \in Processes:
    (p /= q) => ~(in_critical[p] /\ in_critical[q])
```

### Pattern 2: Request-Response
**Liveness:** Every request eventually gets response
```tla
GuaranteedResponse ==
  \A req \in Requests: (req \in sent) ~> (req \in received)
```

### Pattern 3: Eventual Consistency
**Liveness:** Replicas eventually converge
```tla
EventualConsistency ==
  <>[](no_messages =>
       \A r1, r2 \in Replicas: value[r1] = value[r2])
```

### Pattern 4: State Machine Replication
**Safety:** All replicas have consistent log prefixes
```tla
LogConsistency ==
  \A r1, r2 \in Replicas:
    \A i \in 1..Min(Len(log[r1]), Len(log[r2])):
      log[r1][i] = log[r2][i]
```

**Deep dive:** See [references/common-patterns.md](references/common-patterns.md) for complete implementations of mutex, message queues, consensus, leader election, two-phase commit, and more.

## Model Checking with TLC

**TLC** exhaustively explores the state space:

1. Generate all initial states satisfying `Init`
2. Apply all enabled actions from `Next`
3. Check invariants in each state
4. Detect cycles for liveness properties
5. Report violations with concrete traces

**Managing state explosion:**
- Use symmetry reduction
- Bound parameters (queue sizes, process counts)
- Model abstract values instead of concrete data
- Start small, increase gradually

**Counterexamples:**
- **Safety:** Finite trace to violating state
- **Liveness:** Infinite trace with cycle

## Learning Path

1. **Week 1:** Watch Lamport's video lectures (1-3), understand state machines
2. **Week 2:** Write simple specs (counter, mutex), focus on safety properties
3. **Week 3:** Add liveness properties, understand fairness
4. **Week 4:** Model a real system from your codebase, start small

## Common Mistakes

1. Forgetting frame conditions (`UNCHANGED`)
2. Confusing safety and liveness
3. Over-specifying implementation details
4. Not starting with type invariants
5. Making state space too large

**When TLC finds violations:** Read the trace, identify bad state, work backwards to find the action that caused it.

## Integration with Development

**Workflow:** Write Gherkin scenarios → Translate to TLA+ → Run TLC → Fix spec → Implement code → Keep spec as documentation

Store specs in `vault/specifications/formal/` with `.tla` files, traces, and design notes.

## Educational Resources

### Primary Sources

**Leslie Lamport's Materials:**
- [Video Course](https://lamport.azurewebsites.net/video/videos.html) - Start here
- [Specifying Systems](https://lamport.azurewebsites.net/tla/book.html) - Free PDF, comprehensive
- [TLA+ Hyperbook](https://lamport.azurewebsites.net/tla/hyperbook.html) - Interactive learning

### Online Resources

- [Learn TLA+](https://learntla.com/) - Comprehensive online tutorial
- [TLA+ Examples](https://github.com/tlaplus/Examples) - Real-world specifications
- [Hillel Wayne's Blog](https://www.hillelwayne.com/post/tla-messages/) - Practical patterns

### Community Examples

- **Industry specs:** Amazon (S3, DynamoDB), Microsoft (Azure), MongoDB
- **Dr. TLA+ Series:** Practical patterns blog

## Detailed References

For deep dives into specific topics:

- **[references/state-machines.md](references/state-machines.md)** - Init, Next, actions, primed variables, state space, frame conditions, stuttering
- **[references/temporal-logic.md](references/temporal-logic.md)** - Temporal operators (□, ◇, ~>), fairness (WF, SF), combined operators, checking properties
- **[references/invariants-properties.md](references/invariants-properties.md)** - Types of invariants, safety vs liveness, inductive invariants, debugging violations
- **[references/common-patterns.md](references/common-patterns.md)** - Complete implementations: mutex, queues, consensus, leader election, 2PC, read-write locks

## Glossary

**Action** - State transition relating current state to next state (uses primed variables)

**Behavior** - Infinite sequence of states where each transition follows Spec

**Fairness** - Constraint preventing infinite stuttering (WF or SF)

**Invariant** - Boolean expression true in all reachable states (safety property)

**Liveness** - Property that good things eventually happen (uses ◇)

**Primed Variable** - `var'` refers to value in next state

**Safety** - Property that bad things never happen (uses □)

**State** - Complete assignment of values to all variables

**State Space** - Set of all reachable states

**Stuttering** - Transition that doesn't change state

**TLC** - TLA+ model checker that explores state space

## Next Steps

**After understanding these concepts:**

1. **Try recife-modeling.md skill** - Practical Clojure-based TLA+ modeling
2. **Model a real system** - Pick concurrent/distributed code from your codebase
3. **Join TLA+ community** - Google group, Slack, conferences
4. **Read industry specs** - AWS, Azure published specs show real-world complexity

**Remember:** The goal is not perfect specs, but clearer thinking about system behavior.

## Sources

- [Learn TLA+](https://www.learntla.com/core/tla.html)
- [TLA+ Wikipedia](https://en.wikipedia.org/wiki/TLA+)
- [Temporal Logic of Actions - Wikipedia](https://en.wikipedia.org/wiki/Temporal_logic_of_actions)
- [Lamport: Specifying and Verifying Systems](https://lamport.org/pubs/spec-and-verifying.pdf)
- [Temporal Properties - Learn TLA+](https://learntla.com/core/temporal-logic.html)
- [Temporal Operators - Learn TLA+](https://old.learntla.com/temporal-logic/operators/)
- [Hillel Wayne: Safety and Liveness](https://www.hillelwayne.com/post/safety-and-liveness/)
- [Jack Vanlightly: Liveness Properties](https://jack-vanlightly.com/blog/2023/10/10/the-importance-of-liveness-properties-part1)
- [Hillel Wayne: Modeling Message Queues](https://www.hillelwayne.com/post/tla-messages/)
- [TLA+ Examples Repository](https://github.com/tlaplus/Examples)
- [Concurrency in TLA+](https://learntla.com/core/concurrency.html)
- [Lamport Mutex Specification](https://github.com/tlaplus/Examples/blob/master/specifications/lamport_mutex/LamportMutex.tla)
