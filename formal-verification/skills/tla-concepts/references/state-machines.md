# State Machines in TLA+

## Core Concepts

Everything in TLA+ is modeled as a state machine. A specification describes all possible behaviors (sequences of states) that a system can exhibit.

## State

A **state** is a complete snapshot of all variables at a point in time:

```tla
State = {
  balance: 100,
  locked: false,
  owner: "alice",
  status: "active"
}
```

States are assignments of values to variables. The complete system state includes every variable declared in your specification.

## Behaviors

A **behavior** is an infinite sequence of states connected by actions:

```
[State0] --Action1--> [State1] --Action2--> [State2] --Action3--> ...
```

TLA+ specifications describe which behaviors are valid system executions. The model checker (TLC) exhaustively explores all possible behaviors to verify properties.

## Init: Initial States

The `Init` predicate defines which states are valid starting points:

```tla
VARIABLES balance, locked, status

Init ==
  /\ balance = 0
  /\ locked = FALSE
  /\ status = "pending"
```

Multiple initial states are possible if the predicate allows it:

```tla
Init ==
  /\ balance \in {0, 100, 1000}  \* Any of these starting balances
  /\ locked = FALSE
  /\ status = "pending"
```

This models non-deterministic initialization where the system can start in any of these configurations.

## Next: State Transitions

The `Next` action defines all possible state transitions. It's a disjunction (OR) of individual actions:

```tla
Next ==
  \/ Deposit
  \/ Withdraw
  \/ Lock
  \/ Unlock
```

Each sub-action describes a specific way the system can evolve from one state to the next.

## Actions: Defining Transitions

An **action** is a boolean formula relating the current state (unprimed variables) to the next state (primed variables):

```tla
Deposit ==
  /\ locked = FALSE              \* Precondition: must be unlocked
  /\ balance' = balance + 100    \* Effect: increase balance
  /\ UNCHANGED <<locked, status>> \* Frame condition: these don't change
```

### Primed Variables

- `variable` - Value in the current state
- `variable'` - Value in the next state (pronounced "variable prime")

Actions are the core of TLA+: they describe the relationship between consecutive states.

### Example: Complete Action

```tla
Withdraw(amount) ==
  /\ locked = FALSE              \* Precondition
  /\ amount > 0                  \* Input validation
  /\ amount <= balance           \* Sufficient funds check
  /\ balance' = balance - amount \* Update balance
  /\ status' = "active"          \* Update status
  /\ UNCHANGED locked            \* Explicit frame condition
```

This action is **enabled** when all preconditions (unprimed conditions) are true. When enabled, it describes how to compute the next state.

## Frame Conditions

**Critical Rule:** Always specify what stays the same.

Without explicit frame conditions, unmentioned variables can change arbitrarily:

```tla
\* BAD: locked and status could change unpredictably
Deposit == balance' = balance + 100

\* GOOD: Explicit about what doesn't change
Deposit ==
  /\ balance' = balance + 100
  /\ UNCHANGED <<locked, status>>
```

Use `UNCHANGED` for convenience:

```tla
UNCHANGED var           \* Single variable
UNCHANGED <<v1, v2, v3>> \* Multiple variables
```

## Stuttering

A **stuttering step** is a transition where no variables change:

```tla
Stutter == UNCHANGED <<balance, locked, status>>
```

TLA+ allows infinite stuttering by default. This models the fact that your system might stop making progress (crash, deadlock, etc.).

To prevent stuttering, use **fairness constraints** (covered in temporal-logic.md).

## Spec: The Complete Specification

The complete specification combines Init, Next, and temporal properties:

```tla
Spec == Init /\ [][Next]_vars
```

Breaking this down:
- `Init` - Behaviors must start with a valid initial state
- `[][Next]_vars` - Each step is either a Next action or stuttering
- `vars` - The tuple of all variables

The formula `[][Next]_vars` means:
- `[]` - Always (in every step)
- `[Next]_vars` - Either Next is true, or vars don't change (stuttering)

### With Fairness

More realistic specifications prevent infinite stuttering:

```tla
Spec ==
  /\ Init
  /\ [][Next]_vars
  /\ WF_vars(Deposit)    \* Deposit eventually happens if continuously enabled
  /\ WF_vars(Withdraw)   \* Withdraw eventually happens if continuously enabled
```

## State Space

The **state space** is the set of all possible states your system can reach:

```
Initial States
    |
    v
[Reachable States] ---> [More Reachable States] ---> ...
```

TLC explores this space using breadth-first search:
1. Generate all initial states satisfying `Init`
2. For each state, apply all enabled actions from `Next`
3. Generate new states
4. Repeat until no new states are discovered

**State explosion:** The state space can grow exponentially. A system with 10 boolean variables has 2^10 = 1024 possible states. With 20 variables: over 1 million states.

### Managing State Explosion

Techniques to keep state space tractable:

1. **Symmetry reduction:** Treat identical processes as interchangeable
2. **Model values:** Use abstract tokens instead of concrete data
3. **Bounded parameters:** Limit queue sizes, process counts, etc.
4. **Abstractions:** Model only essential algorithm details

## Non-Determinism

TLA+ embraces non-determinism to model:
- Unknown execution order
- Environment inputs
- Implementation choices

```tla
ChooseAction ==
  /\ \E amount \in 1..100:   \* Non-deterministically pick an amount
       balance' = balance + amount
  /\ UNCHANGED <<locked, status>>
```

The model checker explores ALL possible choices. This is how TLA+ finds race conditions and edge cases.

## Example: Complete State Machine

```tla
EXTENDS Integers

VARIABLES balance, locked

vars == <<balance, locked>>

Init ==
  /\ balance = 0
  /\ locked = FALSE

Deposit(amount) ==
  /\ locked = FALSE
  /\ amount > 0
  /\ balance' = balance + amount
  /\ UNCHANGED locked

Withdraw(amount) ==
  /\ locked = FALSE
  /\ amount > 0
  /\ amount <= balance
  /\ balance' = balance - amount
  /\ UNCHANGED locked

Lock ==
  /\ locked = FALSE
  /\ locked' = TRUE
  /\ UNCHANGED balance

Unlock ==
  /\ locked = TRUE
  /\ locked' = FALSE
  /\ UNCHANGED balance

Next ==
  \/ \E amt \in 1..50: Deposit(amt)
  \/ \E amt \in 1..50: Withdraw(amt)
  \/ Lock
  \/ Unlock

Spec == Init /\ [][Next]_vars /\ WF_vars(Unlock)

\* Type invariant
TypeOK ==
  /\ balance \in Nat
  /\ locked \in BOOLEAN

\* Safety invariant
BalanceNonNegative == balance >= 0
```

## Key Takeaways

1. **States** are complete variable assignments
2. **Behaviors** are infinite sequences of states
3. **Init** defines starting states
4. **Next** defines all possible transitions
5. **Actions** use primed variables to relate current and next state
6. **Frame conditions** are mandatory - specify what doesn't change
7. **Stuttering** is allowed by default (system can stop)
8. **Non-determinism** lets TLC explore all possibilities
9. **State space** can explode - use abstractions and bounds

## Further Reading

- Lamport's "Specifying Systems" Chapter 2: [Specifying Systems](https://lamport.azurewebsites.net/tla/book.html)
- Learn TLA+ state machine basics: [Learn TLA+](https://learntla.com/core/tla.html)
- Wikipedia on TLA+: [TLA+ - Wikipedia](https://en.wikipedia.org/wiki/TLA+)
