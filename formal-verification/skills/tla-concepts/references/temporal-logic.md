# Temporal Logic in TLA+

## Overview

Temporal logic extends TLA+ beyond state-based invariants to express properties over entire behaviors (infinite sequences of states). It distinguishes between **safety properties** (bad things don't happen) and **liveness properties** (good things eventually happen).

## Core Temporal Operators

### Always: □ (Box)

**Syntax:** `[]P` or `□P`

**Meaning:** Property P holds in every state of every behavior.

```tla
[]TypeOK  \* Type invariant holds in all states
```

This is equivalent to checking an invariant in TLC. When used on the outside of a predicate, `[]P` means P is true in every reachable state.

**Compositions:**
- `[]P /\ []Q` - Both properties always hold
- `[]P \/ []Q` - At least one property always holds (weaker)
- `[]P => []Q` - If P is always true, then Q is always true

### Eventually: ◇ (Diamond)

**Syntax:** `<>P` or `◇P`

**Meaning:** Property P becomes true in at least one state of every behavior.

```tla
<>terminated  \* Eventually the system terminates
```

Key insight: P only needs to be true momentarily. It can become false again later.

**Definition:** `<>P` is equivalent to `~[]~P` (not always not-P), meaning P must hold somewhere.

### Eventually Always: ◇□

**Syntax:** `<>[]P` or `◇□P`

**Meaning:** Eventually P becomes true and stays true forever.

```tla
<>[]converged  \* Eventually the system converges and stays converged
```

This captures **convergence** properties: the system might be unstable initially, but after some point reaches a stable state.

**Example:** Eventual consistency in distributed systems
```tla
EventualConsistency ==
  <>[](no_pending_messages =>
       \A r1, r2 \in Replicas: value[r1] = value[r2])
```

Once messages are drained, all replicas have the same value forever.

### Always Eventually: □◇

**Syntax:** `[]<>P` or `□◇P`

**Meaning:** At any point in time, P will be true sometime in the future. P is true **infinitely often**.

```tla
[]<>fair_scheduled  \* Process is scheduled infinitely often (never starved)
```

**Example:** Fair scheduling
```tla
FairScheduling ==
  \A p \in Processes: []<>(running[p] = TRUE)
```

Every process runs infinitely often - no process is permanently starved.

### Leads-To: ~>

**Syntax:** `P ~> Q`

**Meaning:** If P becomes true, then Q eventually becomes true afterward (in the current or a future state).

```tla
request_sent ~> response_received  \* Every request gets a response
```

**Definition:** `P ~> Q` is equivalent to `[](P => <>Q)`

This is temporal implication: whenever P becomes true, Q must eventually follow.

**Example:** Request-response guarantee
```tla
RequestResponse ==
  \A req \in Requests: (req \in sent) ~> (req \in received)
```

## Temporal Formula Evaluation

**Critical rule:** Anything outside a temporal operator is tested in the **initial state** only.

```tla
x = 0        \* Only checked in initial state
[](x >= 0)   \* Checked in every state
```

This means:
```tla
Init == x = 5
Spec == x = 0 /\ [][Next]_vars  \* FAILS: x is 5 in initial state, not 0
```

Correct:
```tla
Spec == Init /\ [][Next]_vars
```

## Fairness

Without fairness, TLA+ allows infinite stuttering - the system can stop making progress forever. Fairness constraints force progress.

### Weak Fairness: WF

**Syntax:** `WF_vars(Action)`

**Meaning:** If Action remains **continuously enabled**, it must eventually occur.

```tla
WF_vars(Unlock)  \* If unlocking is always possible, it eventually happens
```

Weak fairness prevents indefinite stalling when an action could always fire.

**Example:**
```tla
Spec == Init /\ [][Next]_vars /\ WF_vars(ProcessMessage)
```

If there are always messages to process, the system must eventually process one.

### Strong Fairness: SF

**Syntax:** `SF_vars(Action)`

**Meaning:** If Action is **repeatedly enabled** (infinitely often), it must eventually occur.

```tla
SF_vars(ReceiveMessage)  \* If messages keep arriving, we eventually receive one
```

Strong fairness is stronger than weak fairness. It's needed when actions might be intermittently disabled.

**Example:** Thread scheduling
```tla
Spec ==
  /\ Init
  /\ [][Next]_vars
  /\ \A thread \in Threads: SF_vars(Schedule(thread))
```

Each thread is eventually scheduled, even if it's sometimes blocked.

### When to Use Each

- **WF:** Use when the action, once enabled, stays enabled until it fires
  - Unlocking a lock
  - Sending an acknowledgment
  - Completing a transaction

- **SF:** Use when the action might be enabled intermittently
  - Receiving network messages
  - Thread scheduling with contention
  - Retrying failed operations

## Combining Operators

Temporal operators compose to express complex properties:

### ◇□P vs □◇P

These look similar but are very different:

**◇□P (Eventually Always):** P becomes permanently true
- "System eventually converges"
- "Errors eventually stop occurring"
- Stronger property

**□◇P (Always Eventually):** P is true infinitely often
- "Heartbeat messages keep arriving"
- "Process gets scheduled infinitely often"
- Weaker property

A property can be both infinitely often true AND infinitely often false. Thus ◇□P is strictly stronger than □◇P.

### Complex Example

```tla
\* If a request is sent, it's eventually received and acknowledged
RequestLifecycle ==
  \A req \in Requests:
    /\ (req \in sent) ~> (req \in received)      \* Request arrives
    /\ (req \in received) ~> (req \in acked)     \* Acknowledgment sent
    /\ [](req \in acked => req \in completed)    \* Acked requests are completed
```

## Common Patterns

### Pattern 1: Eventual Termination

```tla
\* Eventually all work is done and nothing changes
EventualTermination ==
  <>[](UNCHANGED vars)
```

### Pattern 2: Mutual Exclusion with Liveness

```tla
\* Safety: At most one process in critical section
MutexSafety ==
  []\A p, q \in Processes:
    (p /= q) => ~(in_critical[p] /\ in_critical[q])

\* Liveness: Every process that wants to enter eventually does
MutexLiveness ==
  \A p \in Processes:
    (wants_entry[p]) ~> (in_critical[p])
```

### Pattern 3: Eventual Consistency

```tla
\* Eventually all replicas converge (when no messages pending)
EventualConsistency ==
  <>[](no_messages =>
       \A r1, r2 \in Replicas: state[r1] = state[r2])
```

### Pattern 4: Guaranteed Response

```tla
\* Every request eventually gets a response
GuaranteedResponse ==
  \A req \in Requests:
    []((req \in pending) => <>(req \in completed))
```

### Pattern 5: No Deadlock

```tla
\* System always has at least one enabled action
NoDeadlock ==
  []<>TRUE  \* Not just stuttering - progress happens

\* More explicitly:
NoDeadlock ==
  [](\E p \in Processes: ENABLED action[p])
```

## Checking Temporal Properties in TLC

### Invariants (Safety)

Add to model as "Invariants":
```tla
TypeOK
BalanceNonNegative
MutualExclusion
```

TLC checks these in every reachable state. Fast to verify.

### Liveness Properties

Add to model as "Properties":
```tla
EventualTermination
[]<>fair_scheduled
request_sent ~> response_received
```

TLC checks these over entire behaviors. Much slower than invariants.

**Performance tip:** Test safety properties with large models, liveness properties with smaller models.

### Counterexamples

**Safety violation:** TLC provides a finite trace ending in the bad state
```
State 1: balance = 100
State 2: balance = 50
State 3: balance = -10  <-- Invariant violated
```

**Liveness violation:** TLC provides a trace with a cycle showing progress never happens
```
State 1: request_sent = TRUE, received = FALSE
State 2: request_sent = TRUE, received = FALSE
State 3: request_sent = TRUE, received = FALSE
[Back to State 2]  <-- Cycle: response never arrives
```

## Common Mistakes

### Mistake 1: Testing Temporal Properties in Init

```tla
\* WRONG: Only checks initial state
Spec == (balance = 0) /\ [][Next]_vars

\* RIGHT: Use Init predicate
Spec == Init /\ [][Next]_vars
```

### Mistake 2: Confusing ◇□ and □◇

```tla
\* Wrong: Means "infinitely often converged" (can un-converge)
<>[]converged

\* Right: Means "eventually converged forever"
[]<>converged
```

Wait, I had these backwards in the comment! Correct version:

```tla
\* Eventually permanently converged (what you usually want)
<>[]converged

\* Infinitely often converged (weaker - can un-converge)
[]<>converged
```

### Mistake 3: Forgetting Fairness

Without fairness, this liveness property fails:
```tla
request_sent ~> response_received
```

Because the system can stutter forever. Add fairness:
```tla
Spec ==
  /\ Init
  /\ [][Next]_vars
  /\ WF_vars(SendResponse)  \* Must eventually send if possible
```

### Mistake 4: Over-Specifying with Liveness

Don't add liveness constraints that prevent valid behaviors:

```tla
\* Too strong: Requires termination
Spec == Init /\ [][Next]_vars /\ <>[]done

\* Better: Termination is optional but allowed
Spec == Init /\ [][Next]_vars
TerminationOK == <>[]done  \* Separate optional property
```

## Debugging Temporal Properties

When a liveness property fails:

1. **Check fairness:** Is the action that should happen marked as fair?
2. **Check enablement:** Is the action actually enabled in the cycle?
3. **Check cycles:** Where does TLC show the behavior loops?
4. **Add printf debugging:** Use Print() in actions to trace execution
5. **Simplify:** Test with fewer processes/smaller state space

## Further Reading

- Learn TLA+ temporal logic: [Temporal Properties](https://learntla.com/core/temporal-logic.html)
- Hillel Wayne's safety vs liveness: [Safety and Liveness Properties](https://www.hillelwayne.com/post/safety-and-liveness/)
- Lamport's TLA paper: [The Temporal Logic of Actions](https://lamport.azurewebsites.net/pubs/lamport-actions.pdf)
- Temporal operator definitions: [Temporal Operators](https://old.learntla.com/temporal-logic/operators/)
- Stanford Encyclopedia: [Temporal Logic](https://plato.stanford.edu/entries/logic-temporal/)
