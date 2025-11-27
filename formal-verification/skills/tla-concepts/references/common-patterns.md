# Common TLA+ Patterns

## Overview

This document catalogs reusable patterns for modeling common distributed systems and concurrent algorithms. These patterns serve as starting points for your own specifications.

## Pattern 1: Mutual Exclusion

**Problem:** Ensure only one process accesses a shared resource at a time.

**Key Properties:**
- **Safety:** At most one process in critical section
- **Liveness:** Every process that wants to enter eventually does (no starvation)

### Basic Mutex Pattern

```tla
EXTENDS Integers, FiniteSets

CONSTANTS Processes
VARIABLES in_critical, wants_entry

vars == <<in_critical, wants_entry>>

Init ==
  /\ in_critical = [p \in Processes |-> FALSE]
  /\ wants_entry = [p \in Processes |-> FALSE]

RequestEntry(p) ==
  /\ ~in_critical[p]
  /\ wants_entry' = [wants_entry EXCEPT ![p] = TRUE]
  /\ UNCHANGED in_critical

EnterCritical(p) ==
  /\ wants_entry[p]
  /\ ~(\E q \in Processes: (q /= p) /\ in_critical[q])
  /\ in_critical' = [in_critical EXCEPT ![p] = TRUE]
  /\ UNCHANGED wants_entry

ExitCritical(p) ==
  /\ in_critical[p]
  /\ in_critical' = [in_critical EXCEPT ![p] = FALSE]
  /\ wants_entry' = [wants_entry EXCEPT ![p] = FALSE]

Next ==
  \E p \in Processes:
    \/ RequestEntry(p)
    \/ EnterCritical(p)
    \/ ExitCritical(p)

Spec ==
  /\ Init
  /\ [][Next]_vars
  /\ \A p \in Processes: WF_vars(EnterCritical(p))
  /\ \A p \in Processes: WF_vars(ExitCritical(p))

\* Safety: Mutual exclusion
MutexSafety ==
  \A p, q \in Processes:
    (p /= q) => ~(in_critical[p] /\ in_critical[q])

\* Liveness: No starvation
NoStarvation ==
  \A p \in Processes:
    (wants_entry[p]) ~> (in_critical[p])
```

**Real-world examples:**
- Lamport's mutex: [Lamport Mutex](https://github.com/tlaplus/Examples/blob/master/specifications/lamport_mutex/LamportMutex.tla)
- Dijkstra's mutex: [Dijkstra Mutex](https://github.com/jameshfisher/tlaplus/blob/master/examples/dijkstra-mutex/DijkstraMutex.tla)

## Pattern 2: Message Queue

**Problem:** Model asynchronous communication via a message queue.

**Key Properties:**
- **Safety:** Messages delivered in order (if FIFO)
- **Safety:** No message duplication (if exactly-once)
- **Liveness:** Messages eventually delivered

### Simple Queue Pattern

```tla
EXTENDS Sequences, Integers

CONSTANTS Data, MaxQueue
VARIABLES queue

Init == queue = <<>>

\* Type invariant
TypeOK == queue \in Seq(Data)

\* Helper: Append to sequence
seq (+) elem == Append(seq, elem)

\* Writer adds to queue
Write(d) ==
  /\ d \in Data
  /\ Len(queue) < MaxQueue
  /\ queue' = queue (+) d

\* Reader removes from queue
Read ==
  /\ queue /= <<>>
  /\ queue' = Tail(queue)

Next ==
  \/ \E d \in Data: Write(d)
  \/ Read

Spec ==
  /\ Init
  /\ [][Next]_queue
  /\ WF_queue(Read)

\* Safety: Queue never overflows
NoOverflow == Len(queue) <= MaxQueue

\* Liveness: Queue eventually drains
EventuallyEmpty == <>(queue = <<>>)
```

### At-Least-Once Delivery Pattern

Models realistic systems where messages might be duplicated:

```tla
EXTENDS Sequences, Integers

CONSTANTS Writers, Readers, Data, MaxQueue
VARIABLES queues

Init ==
  queues = [r \in Readers |-> <<>>]

\* Writer sends to one reader's queue
Send(w, r, d) ==
  /\ Len(queues[r]) < MaxQueue
  /\ queues' = [queues EXCEPT ![r] = @ (+) d]

\* Reader consumes message (3 possibilities: remove, duplicate, or keep)
Receive(r) ==
  /\ queues[r] /= <<>>
  /\ LET msg == Head(queues[r])
     IN \/ queues' = [queues EXCEPT ![r] = Tail(@)]           \* Normal: remove
        \/ queues' = [queues EXCEPT ![r] = Tail(@) (+) msg]  \* Duplicate
        \/ UNCHANGED queues                                    \* Keep (retry later)

Next ==
  \/ \E w \in Writers, r \in Readers, d \in Data: Send(w, r, d)
  \/ \E r \in Readers: Receive(r)

Spec ==
  /\ Init
  /\ [][Next]_queues
  /\ \A r \in Readers: WF_queues(Receive(r))
```

**Further reading:**
- Hillel Wayne's message queue tutorial: [Modeling Message Queues](https://www.hillelwayne.com/post/tla-messages/)

## Pattern 3: Distributed Consensus

**Problem:** Multiple processes agree on a single value.

**Key Properties (FLP consensus):**
- **Safety - Agreement:** No two processes decide different values
- **Safety - Validity:** Any decided value was proposed
- **Liveness - Termination:** All processes eventually decide

### Simple Consensus Pattern

```tla
EXTENDS Integers, FiniteSets

CONSTANTS Processes, Values
VARIABLES proposed, decided

vars == <<proposed, decided>>

Init ==
  /\ proposed \in SUBSET Values  \* Non-deterministic initial proposals
  /\ proposed /= {}              \* At least one proposal
  /\ decided = [p \in Processes |-> NULL]

Propose(p, v) ==
  /\ v \in Values
  /\ proposed' = proposed \union {v}
  /\ UNCHANGED decided

Decide(p, v) ==
  /\ decided[p] = NULL
  /\ v \in proposed
  /\ decided' = [decided EXCEPT ![p] = v]
  /\ UNCHANGED proposed

Next ==
  \/ \E p \in Processes, v \in Values: Propose(p, v)
  \/ \E p \in Processes, v \in Values: Decide(p, v)

Spec ==
  /\ Init
  /\ [][Next]_vars
  /\ \A p \in Processes: WF_vars(Decide(p, CHOOSE v \in proposed: TRUE))

\* Safety: Agreement
Agreement ==
  \A p1, p2 \in Processes:
    (decided[p1] /= NULL /\ decided[p2] /= NULL)
    => (decided[p1] = decided[p2])

\* Safety: Validity
Validity ==
  \A p \in Processes:
    (decided[p] /= NULL) => (decided[p] \in proposed)

\* Liveness: Termination
Termination ==
  <>(\A p \in Processes: decided[p] /= NULL)
```

**Real implementations:**
- Raft consensus
- Paxos
- Byzantine consensus algorithms

## Pattern 4: Eventual Consistency

**Problem:** Replicas converge to the same state after updates stop.

**Key Properties:**
- **Safety:** No conflicting updates
- **Liveness:** Eventually all replicas agree

### Eventual Consistency Pattern

```tla
EXTENDS Integers, FiniteSets

CONSTANTS Replicas, Values
VARIABLES replica_value, messages

vars == <<replica_value, messages>>

Init ==
  /\ replica_value = [r \in Replicas |-> 0]
  /\ messages = {}

\* Client updates a replica
Update(r, v) ==
  /\ v \in Values
  /\ replica_value' = [replica_value EXCEPT ![r] = v]
  /\ messages' = messages \union
       {[from |-> r, to |-> dest, value |-> v] : dest \in Replicas \ {r}}

\* Replica receives and applies update
ReceiveUpdate ==
  /\ messages /= {}
  /\ \E msg \in messages:
       /\ replica_value' = [replica_value EXCEPT ![msg.to] = msg.value]
       /\ messages' = messages \ {msg}

Next ==
  \/ \E r \in Replicas, v \in Values: Update(r, v)
  \/ ReceiveUpdate

Spec ==
  /\ Init
  /\ [][Next]_vars
  /\ WF_vars(ReceiveUpdate)

\* Liveness: Eventually consistent
EventualConsistency ==
  <>[](messages = {} =>
       \A r1, r2 \in Replicas: replica_value[r1] = replica_value[r2])
```

## Pattern 5: State Machine Replication

**Problem:** Multiple replicas execute operations in the same order.

**Key Properties:**
- **Safety:** All replicas have consistent log prefixes
- **Liveness:** Operations eventually appear in all logs

### Log Replication Pattern

```tla
EXTENDS Sequences, Integers

CONSTANTS Replicas, Operations
VARIABLES log, committed

vars == <<log, committed>>

Init ==
  /\ log = [r \in Replicas |-> <<>>]
  /\ committed = 0  \* Index of last committed entry

\* Leader appends operation to its log
Append(r, op) ==
  /\ op \in Operations
  /\ log' = [log EXCEPT ![r] = Append(@, op)]
  /\ UNCHANGED committed

\* Replicate entry to follower
Replicate(leader, follower, index) ==
  /\ index <= Len(log[leader])
  /\ Len(log[follower]) < index
  /\ log' = [log EXCEPT ![follower] =
       Append(@, log[leader][index])]
  /\ UNCHANGED committed

\* Commit an entry when majority have it
Commit(index) ==
  /\ index > committed
  /\ LET replicas_with_entry ==
         {r \in Replicas : Len(log[r]) >= index}
     IN Cardinality(replicas_with_entry) > Cardinality(Replicas) \div 2
  /\ committed' = index
  /\ UNCHANGED log

Next ==
  \/ \E r \in Replicas, op \in Operations: Append(r, op)
  \/ \E leader, follower \in Replicas, idx \in 1..10: Replicate(leader, follower, idx)
  \/ \E idx \in 1..10: Commit(idx)

Spec ==
  /\ Init
  /\ [][Next]_vars
  /\ \A r1, r2 \in Replicas: WF_vars(Replicate(r1, r2, committed + 1))

\* Safety: Log consistency
LogConsistency ==
  \A r1, r2 \in Replicas:
    \A i \in 1..Min(Len(log[r1]), Len(log[r2])):
      log[r1][i] = log[r2][i]
```

## Pattern 6: Two-Phase Commit

**Problem:** Atomic commitment across multiple participants.

**Key Properties:**
- **Safety:** If one commits, all commit; if one aborts, none commit
- **Liveness:** Eventually decide (commit or abort)

### Two-Phase Commit Pattern

```tla
EXTENDS FiniteSets

CONSTANTS Participants, Coordinator
VARIABLES state, votes, decision

vars == <<state, votes, decision>>

Init ==
  /\ state = [p \in Participants |-> "working"]
  /\ votes = {}
  /\ decision = "undecided"

\* Phase 1: Participant votes
Vote(p, vote) ==
  /\ state[p] = "working"
  /\ vote \in {"yes", "no"}
  /\ votes' = votes \union {[participant |-> p, vote |-> vote]}
  /\ state' = [state EXCEPT ![p] = "voted"]
  /\ UNCHANGED decision

\* Phase 2: Coordinator decides based on votes
Decide ==
  /\ decision = "undecided"
  /\ \A p \in Participants: \E v \in votes: v.participant = p
  /\ LET all_yes == \A v \in votes: v.vote = "yes"
     IN decision' = IF all_yes THEN "commit" ELSE "abort"
  /\ UNCHANGED <<state, votes>>

\* Participants learn decision
Learn(p) ==
  /\ decision /= "undecided"
  /\ state[p] = "voted"
  /\ state' = [state EXCEPT ![p] = decision]
  /\ UNCHANGED <<votes, decision>>

Next ==
  \/ \E p \in Participants: Vote(p, "yes")
  \/ \E p \in Participants: Vote(p, "no")
  \/ Decide
  \/ \E p \in Participants: Learn(p)

Spec ==
  /\ Init
  /\ [][Next]_vars
  /\ WF_vars(Decide)
  /\ \A p \in Participants: WF_vars(Learn(p))

\* Safety: All participants reach same decision
AtomicCommit ==
  \A p1, p2 \in Participants:
    (state[p1] \in {"commit", "abort"} /\ state[p2] \in {"commit", "abort"})
    => (state[p1] = state[p2])
```

## Pattern 7: Leader Election

**Problem:** Elect exactly one leader among multiple processes.

**Key Properties:**
- **Safety:** At most one leader at any time
- **Liveness:** Eventually a leader is elected

### Simple Leader Election Pattern

```tla
EXTENDS Integers, FiniteSets

CONSTANTS Processes
VARIABLES leader, candidates

vars == <<leader, candidates>>

Init ==
  /\ leader = NULL
  /\ candidates = {}

\* Process nominates itself
Nominate(p) ==
  /\ p \in Processes
  /\ candidates' = candidates \union {p}
  /\ UNCHANGED leader

\* Elect leader (highest ID, for simplicity)
Elect ==
  /\ leader = NULL
  /\ candidates /= {}
  /\ leader' = CHOOSE p \in candidates:
       \A q \in candidates: p >= q
  /\ UNCHANGED candidates

\* Leader failure (loses leadership)
FailLeader ==
  /\ leader /= NULL
  /\ leader' = NULL
  /\ candidates' = {}

Next ==
  \/ \E p \in Processes: Nominate(p)
  \/ Elect
  \/ FailLeader

Spec ==
  /\ Init
  /\ [][Next]_vars
  /\ WF_vars(Elect)

\* Safety: At most one leader
SingleLeader ==
  (leader /= NULL) => (leader \in Processes)

\* Liveness: Eventually a leader exists
EventualLeader ==
  <>(leader /= NULL)
```

## Pattern 8: Producer-Consumer

**Problem:** Producers add items, consumers remove them, with bounded buffer.

**Key Properties:**
- **Safety:** Buffer never overflows or underflows
- **Liveness:** Items eventually consumed

### Producer-Consumer Pattern

```tla
EXTENDS Sequences, Integers

CONSTANTS Producers, Consumers, Items, BufSize
VARIABLES buffer

Init == buffer = <<>>

Produce(p, item) ==
  /\ p \in Producers
  /\ item \in Items
  /\ Len(buffer) < BufSize
  /\ buffer' = Append(buffer, item)

Consume(c) ==
  /\ c \in Consumers
  /\ buffer /= <<>>
  /\ buffer' = Tail(buffer)

Next ==
  \/ \E p \in Producers, item \in Items: Produce(p, item)
  \/ \E c \in Consumers: Consume(c)

Spec ==
  /\ Init
  /\ [][Next]_buffer
  /\ \A c \in Consumers: WF_buffer(Consume(c))

\* Safety: Buffer bounded
BufferBounded ==
  Len(buffer) <= BufSize

\* Liveness: Buffer eventually drains
EventuallyEmpty ==
  <>[]( buffer = <<>> )
```

## Pattern 9: Read-Write Lock

**Problem:** Multiple readers or single writer can access shared resource.

**Key Properties:**
- **Safety:** Readers and writers mutually excluded
- **Safety:** Multiple readers allowed simultaneously
- **Liveness:** No writer or reader starvation

### Read-Write Lock Pattern

```tla
EXTENDS Integers, FiniteSets

CONSTANTS Readers, Writers
VARIABLES reading, writing, waiting_readers, waiting_writers

vars == <<reading, writing, waiting_readers, waiting_writers>>

Init ==
  /\ reading = {}
  /\ writing = NULL
  /\ waiting_readers = {}
  /\ waiting_writers = {}

RequestRead(r) ==
  /\ r \in Readers
  /\ r \notin reading
  /\ waiting_readers' = waiting_readers \union {r}
  /\ UNCHANGED <<reading, writing, waiting_writers>>

AcquireRead(r) ==
  /\ r \in waiting_readers
  /\ writing = NULL
  /\ reading' = reading \union {r}
  /\ waiting_readers' = waiting_readers \ {r}
  /\ UNCHANGED <<writing, waiting_writers>>

ReleaseRead(r) ==
  /\ r \in reading
  /\ reading' = reading \ {r}
  /\ UNCHANGED <<writing, waiting_readers, waiting_writers>>

RequestWrite(w) ==
  /\ w \in Writers
  /\ writing /= w
  /\ waiting_writers' = waiting_writers \union {w}
  /\ UNCHANGED <<reading, writing, waiting_readers>>

AcquireWrite(w) ==
  /\ w \in waiting_writers
  /\ reading = {}
  /\ writing = NULL
  /\ writing' = w
  /\ waiting_writers' = waiting_writers \ {w}
  /\ UNCHANGED <<reading, waiting_readers>>

ReleaseWrite(w) ==
  /\ writing = w
  /\ writing' = NULL
  /\ UNCHANGED <<reading, waiting_readers, waiting_writers>>

Next ==
  \/ \E r \in Readers: RequestRead(r) \/ AcquireRead(r) \/ ReleaseRead(r)
  \/ \E w \in Writers: RequestWrite(w) \/ AcquireWrite(w) \/ ReleaseWrite(w)

Spec ==
  /\ Init
  /\ [][Next]_vars
  /\ \A r \in Readers: WF_vars(AcquireRead(r))
  /\ \A w \in Writers: WF_vars(AcquireWrite(w))

\* Safety: Readers and writers mutually excluded
RWMutex ==
  /\ (reading /= {}) => (writing = NULL)
  /\ (writing /= NULL) => (reading = {})
```

## Reusable Helpers

### Sequence Helpers

```tla
\* Append operator (more readable)
seq (+) elem == Append(seq, elem)

\* Remove element at index
Remove(seq, index) ==
  SubSeq(seq, 1, index-1) \o SubSeq(seq, index+1, Len(seq))

\* Check if sequence contains element
Contains(seq, elem) ==
  \E i \in 1..Len(seq): seq[i] = elem
```

### Set Helpers

```tla
\* Minimum of a set
Min(S) == CHOOSE x \in S: \A y \in S: x <= y

\* Maximum of a set
Max(S) == CHOOSE x \in S: \A y \in S: x >= y

\* Majority of replicas
Majority(replicas) == (Cardinality(replicas) \div 2) + 1
```

## Further Examples

**TLA+ Examples Repository:**
- [TLA+ Examples on GitHub](https://github.com/tlaplus/Examples)
- Comprehensive collection covering applications in various domains

**Specific Examples:**
- Lamport's distributed mutex: [Lamport Mutex](https://github.com/tlaplus/Examples/blob/master/specifications/lamport_mutex/LamportMutex.tla)
- Message queue modeling: [Modeling Message Queues](https://www.hillelwayne.com/post/tla-messages/)
- Concurrency patterns: [Concurrency in TLA+](https://learntla.com/core/concurrency.html)
- Distributed locking: [Modelling Distributed Locking](https://medium.com/@polyglot_factotum/modelling-distributed-locking-in-tla-8a75dc441c5a)

**Industry Specifications:**
- Amazon S3, DynamoDB
- Microsoft Azure services
- MongoDB replication
