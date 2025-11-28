---
name: tla-concepts
description: Use when user asks about formal verification, TLA+ syntax, temporal logic, invariants, or how to specify system correctness properties.
requires:
  tools: []
  skills: []
---

# TLA+ Concepts

TLA+ (Temporal Logic of Actions) formally specifies and verifies concurrent/distributed systems by exhaustively checking all possible behaviors.

## When to Use

- Concurrency (race conditions, locks, consensus)
- Correctness-critical (financial, safety, security)
- Before implementation (catch design flaws early)

## Core Concepts

### State Machine

```tla
VARIABLES balance, locked

Init == balance = 0 /\ locked = FALSE

Deposit ==
  /\ locked = FALSE
  /\ balance' = balance + 100
  /\ locked' = locked

Next == Deposit \/ Withdraw \/ ...

Spec == Init /\ [][Next]_<<balance, locked>>
```

- **State**: Snapshot of all variables
- **Init**: Valid starting states
- **Next**: All possible transitions
- **Primed (')**: Next-state value

### Invariants (Safety)

Properties that must ALWAYS hold:
```tla
TypeInvariant == balance \in Int /\ locked \in BOOLEAN

BalanceNonNegative == balance >= 0
```

### Temporal Properties (Liveness)

Properties about system progress:
```tla
\* Eventually responds
EventuallyResponds == request ~> response

\* Always eventually unlocked
AlwaysEventuallyUnlocked == []<>(locked = FALSE)
```

## Temporal Operators

| Operator | Meaning |
|----------|---------|
| `[]P` | Always P (safety) |
| `<>P` | Eventually P |
| `P ~> Q` | P leads to Q |
| `[]<>P` | Infinitely often P |
| `<>[]P` | Eventually always P |

## Model Checking

TLC exhaustively explores all states:
```bash
tlc Spec.tla -config Spec.cfg
```

**Finds**:
- Invariant violations (counterexample trace)
- Deadlocks (no valid next state)
- Liveness violations (infinite loops without progress)

## Quick Syntax

```tla
\* Logical
/\ (and), \/ (or), ~ (not), => (implies)

\* Sets
\in (member), \subseteq (subset), UNION, SUBSET

\* Functions
[x \in S |-> expr]  \* Function definition
f[x]                \* Function application

\* Quantifiers
\A x \in S : P(x)   \* For all
\E x \in S : P(x)   \* Exists
```

## Translation to Recife

Use `recife-modeling` skill to write TLA+ specs in Clojure:
- `r/definit` = Init
- `r/defaction` = Actions
- `r/definvariant` = Invariants
- `r/defproperty` = Temporal properties

## Success Criteria

- [ ] User understands state machine model (Init, Next, invariants)
- [ ] Temporal operators explained with examples
- [ ] Connection to Recife/Clojure made clear

## Related Skills

- `recife-modeling` - TLA+ in Clojure syntax
