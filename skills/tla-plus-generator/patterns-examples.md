# TLA+ Patterns and Examples

This document contains comprehensive examples of common TLA+ specification patterns.

## Example 1: Key-Value Store with Snapshot Isolation

A concurrent key-value store with transactional semantics:

```tla
--------------------------- MODULE KeyValueStore ---------------------------
CONSTANTS   Key,            \* The set of all keys
            Val,            \* The set of all values
            TxId            \* The set of all transaction IDs

VARIABLES   store,          \* Data store mapping keys to values
            tx,             \* Set of open snapshot transactions
            snapshotStore,  \* Snapshots of store for each transaction
            written,        \* Log of writes within each transaction
            missed          \* Writes invisible to each transaction
----------------------------------------------------------------------------

NoVal == CHOOSE v : v \notin Val

Store == [Key -> Val \cup {NoVal}]

Init ==
    /\ store = [k \in Key |-> NoVal]
    /\ tx = {}
    /\ snapshotStore = [t \in TxId |-> [k \in Key |-> NoVal]]
    /\ written = [t \in TxId |-> {}]
    /\ missed = [t \in TxId |-> {}]

TypeInvariant ==
    /\ store \in Store
    /\ tx \subseteq TxId
    /\ snapshotStore \in [TxId -> Store]
    /\ written \in [TxId -> SUBSET Key]
    /\ missed \in [TxId -> SUBSET Key]

OpenTx(t) ==
    /\ t \notin tx
    /\ tx' = tx \cup {t}
    /\ snapshotStore' = [snapshotStore EXCEPT ![t] = store]
    /\ UNCHANGED <<written, missed, store>>

Add(t, k, v) ==
    /\ t \in tx
    /\ snapshotStore[t][k] = NoVal
    /\ snapshotStore' = [snapshotStore EXCEPT ![t][k] = v]
    /\ written' = [written EXCEPT ![t] = @ \cup {k}]
    /\ UNCHANGED <<tx, missed, store>>

CloseTx(t) ==
    /\ t \in tx
    /\ missed[t] \cap written[t] = {}   \* No write-write conflicts
    /\ store' = [k \in Key |->
        IF k \in written[t] THEN snapshotStore[t][k] ELSE store[k]]
    /\ tx' = tx \ {t}
    /\ missed' = [otherTx \in TxId |->
        IF otherTx \in tx' THEN missed[otherTx] \cup written[t] ELSE {}]
    /\ snapshotStore' = [snapshotStore EXCEPT ![t] = [k \in Key |-> NoVal]]
    /\ written' = [written EXCEPT ![t] = {}]

Next ==
    \/ \E t \in TxId : OpenTx(t)
    \/ \E t \in tx : \E k \in Key : \E v \in Val : Add(t, k, v)
    \/ \E t \in tx : CloseTx(t)

Spec == Init /\ [][Next]_<<store, tx, snapshotStore, written, missed>>
=============================================================================
```

## Example 2: Mutual Exclusion (Bakery Algorithm)

Lamport's bakery algorithm for mutual exclusion:

```tla
---------------------------- MODULE Bakery ----------------------------
EXTENDS Naturals

CONSTANT N
ASSUME N \in Nat

Procs == 1..N

\* Lexicographic ordering on pairs
a \prec b == \/ a[1] < b[1]
             \/ (a[1] = b[1]) /\ (a[2] < b[2])

VARIABLES num, flag, pc

vars == <<num, flag, pc>>

Init ==
    /\ num = [i \in Procs |-> 0]
    /\ flag = [i \in Procs |-> FALSE]
    /\ pc = [i \in Procs |-> "ncs"]

\* Non-critical section
ncs(self) ==
    /\ pc[self] = "ncs"
    /\ pc' = [pc EXCEPT ![self] = "e1"]
    /\ UNCHANGED <<num, flag>>

\* Entry protocol: set flag and get ticket number
e1(self) ==
    /\ pc[self] = "e1"
    /\ flag' = [flag EXCEPT ![self] = TRUE]
    /\ num' = [num EXCEPT ![self] =
        1 + CHOOSE m \in Nat : \A i \in Procs : num[i] <= m]
    /\ pc' = [pc EXCEPT ![self] = "e2"]

\* Entry protocol: clear flag
e2(self) ==
    /\ pc[self] = "e2"
    /\ flag' = [flag EXCEPT ![self] = FALSE]
    /\ pc' = [pc EXCEPT ![self] = "w1"]
    /\ UNCHANGED num

\* Wait for others with lower numbers
w1(self) ==
    /\ pc[self] = "w1"
    /\ \A j \in Procs \ {self} :
        \/ num[j] = 0
        \/ <<num[self], self>> \prec <<num[j], j>>
    /\ pc' = [pc EXCEPT ![self] = "cs"]
    /\ UNCHANGED <<num, flag>>

\* Critical section
cs(self) ==
    /\ pc[self] = "cs"
    /\ pc' = [pc EXCEPT ![self] = "exit"]
    /\ UNCHANGED <<num, flag>>

\* Exit: reset number
exit(self) ==
    /\ pc[self] = "exit"
    /\ num' = [num EXCEPT ![self] = 0]
    /\ pc' = [pc EXCEPT ![self] = "ncs"]
    /\ UNCHANGED flag

p(self) == ncs(self) \/ e1(self) \/ e2(self) \/ w1(self) \/ cs(self) \/ exit(self)

Next == \E self \in Procs : p(self)

Spec == Init /\ [][Next]_vars /\ \A self \in Procs : WF_vars(p(self))

\* Safety: mutual exclusion
MutualExclusion == \A i, j \in Procs :
    (i /= j) => ~(pc[i] = "cs" /\ pc[j] = "cs")

\* Liveness: starvation freedom
StarvationFree == \A i \in Procs : pc[i] = "e1" ~> pc[i] = "cs"

TypeOK ==
    /\ num \in [Procs -> Nat]
    /\ flag \in [Procs -> BOOLEAN]
    /\ pc \in [Procs -> {"ncs", "e1", "e2", "w1", "cs", "exit"}]
=============================================================================
```

## Example 3: Producer-Consumer with Bounded Buffer

```tla
------------------------ MODULE ProducerConsumer ------------------------
EXTENDS Naturals, Sequences

CONSTANTS Producers, Consumers, BufferSize, Data

VARIABLES buffer, prodState, consState

vars == <<buffer, prodState, consState>>

TypeInvariant ==
    /\ buffer \in Seq(Data)
    /\ Len(buffer) <= BufferSize
    /\ prodState \in [Producers -> {"idle", "producing"}]
    /\ consState \in [Consumers -> {"idle", "consuming"}]

Init ==
    /\ buffer = <<>>
    /\ prodState = [p \in Producers |-> "idle"]
    /\ consState = [c \in Consumers |-> "idle"]

Produce(p, d) ==
    /\ prodState[p] = "idle"
    /\ Len(buffer) < BufferSize
    /\ buffer' = Append(buffer, d)
    /\ prodState' = [prodState EXCEPT ![p] = "producing"]
    /\ UNCHANGED consState

FinishProducing(p) ==
    /\ prodState[p] = "producing"
    /\ prodState' = [prodState EXCEPT ![p] = "idle"]
    /\ UNCHANGED <<buffer, consState>>

Consume(c) ==
    /\ consState[c] = "idle"
    /\ Len(buffer) > 0
    /\ buffer' = Tail(buffer)
    /\ consState' = [consState EXCEPT ![c] = "consuming"]
    /\ UNCHANGED prodState

FinishConsuming(c) ==
    /\ consState[c] = "consuming"
    /\ consState' = [consState EXCEPT ![c] = "idle"]
    /\ UNCHANGED <<buffer, prodState>>

Next ==
    \/ \E p \in Producers, d \in Data : Produce(p, d)
    \/ \E p \in Producers : FinishProducing(p)
    \/ \E c \in Consumers : Consume(c)
    \/ \E c \in Consumers : FinishConsuming(c)

Spec == Init /\ [][Next]_vars

FairSpec == Spec
    /\ \A p \in Producers : WF_vars(\E d \in Data : Produce(p, d))
    /\ \A c \in Consumers : WF_vars(Consume(c))

\* Safety: buffer never overflows or underflows
BufferSafety == Len(buffer) >= 0 /\ Len(buffer) <= BufferSize

\* Liveness: items eventually get consumed
Progress == \A d \in Data :
    (d \in {buffer[i] : i \in 1..Len(buffer)}) ~>
    (d \notin {buffer[i] : i \in 1..Len(buffer)})
=============================================================================
```

## Example 4: Distributed Spanning Tree (Echo Algorithm)

```tla
-------------------------------- MODULE Echo --------------------------------
EXTENDS Naturals, FiniteSets

CONSTANTS Node, initiator, R  \* R is adjacency relation

ASSUME /\ initiator \in Node
       /\ R \in [Node \X Node -> BOOLEAN]

NoNode == CHOOSE x : x \notin Node
neighbors(n) == {m \in Node : R[m, n]}

VARIABLES inbox, parent, children, rcvd

vars == <<inbox, parent, children, rcvd>>

\* Network operations
send(net, p, q, knd) == [net EXCEPT ![q] = @ \cup {[kind |-> knd, sndr |-> p]}]
receive(net, p, msg) == [net EXCEPT ![p] = @ \ {msg}]
multicast(net, p, dest, knd) ==
    [m \in Node |-> IF m \in dest
                    THEN net[m] \cup {[kind |-> knd, sndr |-> p]}
                    ELSE net[m]]

Init ==
    /\ inbox = [n \in Node |-> {}]
    /\ parent = [n \in Node |-> NoNode]
    /\ children = [n \in Node |-> {}]
    /\ rcvd = [n \in Node |-> 0]

\* Initiator starts the algorithm
InitiatorStart ==
    /\ rcvd[initiator] = 0
    /\ inbox' = multicast(inbox, initiator, neighbors(initiator), "m")
    /\ rcvd' = [rcvd EXCEPT ![initiator] = Cardinality(neighbors(initiator))]
    /\ UNCHANGED <<parent, children>>

\* Non-initiator receives first message
FirstReceive(n) ==
    /\ n /= initiator
    /\ rcvd[n] = 0
    /\ \E msg \in inbox[n] :
        /\ msg.kind = "m"
        /\ parent' = [parent EXCEPT ![n] = msg.sndr]
        /\ inbox' = multicast(receive(inbox, n, msg), n,
                              neighbors(n) \ {msg.sndr}, "m")
        /\ rcvd' = [rcvd EXCEPT ![n] = 1]
    /\ UNCHANGED children

\* Receive subsequent messages
SubsequentReceive(n) ==
    /\ rcvd[n] > 0
    /\ rcvd[n] < Cardinality(neighbors(n))
    /\ \E msg \in inbox[n] :
        /\ inbox' = receive(inbox, n, msg)
        /\ rcvd' = [rcvd EXCEPT ![n] = @ + 1]
        /\ IF msg.kind = "c"
           THEN children' = [children EXCEPT ![n] = @ \cup {msg.sndr}]
           ELSE UNCHANGED children
    /\ UNCHANGED parent

\* Send acknowledgment to parent
SendAck(n) ==
    /\ n /= initiator
    /\ rcvd[n] = Cardinality(neighbors(n))
    /\ parent[n] /= NoNode
    /\ inbox' = send(inbox, n, parent[n], "c")
    /\ parent' = [parent EXCEPT ![n] = NoNode]  \* Mark as done
    /\ UNCHANGED <<children, rcvd>>

Next ==
    \/ InitiatorStart
    \/ \E n \in Node : FirstReceive(n)
    \/ \E n \in Node : SubsequentReceive(n)
    \/ \E n \in Node : SendAck(n)

Spec == Init /\ [][Next]_vars

\* The initiator never has a parent
InitiatorNoParent == parent[initiator] = NoNode

\* At termination, initiator has all children in spanning tree
SpanningTree ==
    (\A n \in Node : rcvd[n] = Cardinality(neighbors(n))) =>
    \A n \in Node \ {initiator} :
        \E path \in Seq(Node) :
            /\ path[1] = n
            /\ path[Len(path)] = initiator
=============================================================================
```

## Example 5: Multi-Car Elevator System

```tla
------------------------------ MODULE Elevator ------------------------------
EXTENDS Integers

CONSTANTS Person, Elevator, FloorCount

VARIABLES PersonState, ActiveElevatorCalls, ElevatorState

Vars == <<PersonState, ActiveElevatorCalls, ElevatorState>>

Floor == 1..FloorCount
Direction == {"Up", "Down"}
ElevatorCall == [floor : Floor, direction : Direction]
ElevatorDirectionState == Direction \cup {"Stationary"}

GetDirection[current, destination \in Floor] ==
    IF destination > current THEN "Up" ELSE "Down"

GetDistance[f1, f2 \in Floor] ==
    IF f1 > f2 THEN f1 - f2 ELSE f2 - f1

CanServiceCall[e \in Elevator, c \in ElevatorCall] ==
    LET eState == ElevatorState[e] IN
    /\ c.floor = eState.floor
    /\ c.direction = eState.direction

TypeInvariant ==
    /\ PersonState \in [Person -> [location : Floor \cup Elevator,
                                   destination : Floor,
                                   waiting : BOOLEAN]]
    /\ ActiveElevatorCalls \subseteq ElevatorCall
    /\ ElevatorState \in [Elevator -> [floor : Floor,
                                       direction : ElevatorDirectionState,
                                       doorsOpen : BOOLEAN,
                                       buttonsPressed : SUBSET Floor]]

Init ==
    /\ PersonState \in [Person -> [location : Floor,
                                   destination : Floor,
                                   waiting : {FALSE}]]
    /\ ActiveElevatorCalls = {}
    /\ ElevatorState \in [Elevator -> [floor : Floor,
                                       direction : {"Stationary"},
                                       doorsOpen : {FALSE},
                                       buttonsPressed : {{}}]]

CallElevator(p) ==
    LET
        pState == PersonState[p]
        call == [floor |-> pState.location,
                 direction |-> GetDirection[pState.location, pState.destination]]
    IN
    /\ ~pState.waiting
    /\ pState.location /= pState.destination
    /\ pState.location \in Floor
    /\ ActiveElevatorCalls' = ActiveElevatorCalls \cup {call}
    /\ PersonState' = [PersonState EXCEPT ![p] = [@ EXCEPT !.waiting = TRUE]]
    /\ UNCHANGED ElevatorState

OpenDoors(e) ==
    LET eState == ElevatorState[e] IN
    /\ ~eState.doorsOpen
    /\ \/ \E call \in ActiveElevatorCalls : CanServiceCall[e, call]
       \/ eState.floor \in eState.buttonsPressed
    /\ ElevatorState' = [ElevatorState EXCEPT ![e] =
        [@ EXCEPT !.doorsOpen = TRUE,
                  !.buttonsPressed = @ \ {eState.floor}]]
    /\ ActiveElevatorCalls' = ActiveElevatorCalls \
        {[floor |-> eState.floor, direction |-> eState.direction]}
    /\ UNCHANGED PersonState

MoveElevator(e) ==
    LET
        eState == ElevatorState[e]
        nextFloor == IF eState.direction = "Up"
                     THEN eState.floor + 1
                     ELSE eState.floor - 1
    IN
    /\ eState.direction /= "Stationary"
    /\ ~eState.doorsOpen
    /\ nextFloor \in Floor
    /\ ElevatorState' = [ElevatorState EXCEPT ![e] =
        [@ EXCEPT !.floor = nextFloor]]
    /\ UNCHANGED <<PersonState, ActiveElevatorCalls>>

Next ==
    \/ \E p \in Person : CallElevator(p)
    \/ \E e \in Elevator : OpenDoors(e)
    \/ \E e \in Elevator : MoveElevator(e)

Spec == Init /\ [][Next]_Vars

\* Safety: elevator doors only open at valid floors
DoorsOpenAtValidFloor ==
    \A e \in Elevator : ElevatorState[e].doorsOpen =>
        ElevatorState[e].floor \in Floor

\* Liveness: every call eventually serviced
CallsServiced == \A c \in ElevatorCall :
    c \in ActiveElevatorCalls ~> \E e \in Elevator : CanServiceCall[e, c]
=============================================================================
```

## Example 6: Cigarette Smokers Problem

Classic synchronization problem:

```tla
-------------------------- MODULE CigaretteSmokers --------------------------
EXTENDS Integers, FiniteSets

CONSTANT Ingredients, Offers

VARIABLE smokers, dealer

ASSUME /\ Offers \subseteq (SUBSET Ingredients)
       /\ \A n \in Offers : Cardinality(n) = Cardinality(Ingredients) - 1

TypeOK ==
    /\ smokers \in [Ingredients -> [smoking: BOOLEAN]]
    /\ dealer \in Offers \/ dealer = {}

vars == <<smokers, dealer>>

ChooseOne(S, P(_)) == CHOOSE x \in S : P(x) /\ \A y \in S : P(y) => y = x

Init ==
    /\ smokers = [r \in Ingredients |-> [smoking |-> FALSE]]
    /\ dealer \in Offers

startSmoking ==
    /\ dealer /= {}
    /\ smokers' = [r \in Ingredients |->
        [smoking |-> {r} \cup dealer = Ingredients]]
    /\ dealer' = {}

stopSmoking ==
    /\ dealer = {}
    /\ LET r == ChooseOne(Ingredients, LAMBDA x : smokers[x].smoking)
       IN smokers' = [smokers EXCEPT ![r].smoking = FALSE]
    /\ dealer' \in Offers

Next == startSmoking \/ stopSmoking

Spec == Init /\ [][Next]_vars
FairSpec == Spec /\ WF_vars(Next)

\* At most one smoker at a time
AtMostOne == Cardinality({r \in Ingredients : smokers[r].smoking}) <= 1
=============================================================================
```

## Example 7: Simple Consensus Protocol

```tla
---------------------------- MODULE Consensus ----------------------------
EXTENDS Naturals, FiniteSets

CONSTANTS Value, Acceptor, Quorum

ASSUME /\ \A Q \in Quorum : Q \subseteq Acceptor
       /\ \A Q1, Q2 \in Quorum : Q1 \cap Q2 /= {}

VARIABLES votes, decision

vars == <<votes, decision>>

TypeOK ==
    /\ votes \in [Acceptor -> SUBSET Value]
    /\ decision \in SUBSET Value

Init ==
    /\ votes = [a \in Acceptor |-> {}]
    /\ decision = {}

Vote(a, v) ==
    /\ votes[a] = {}  \* Each acceptor votes once
    /\ votes' = [votes EXCEPT ![a] = {v}]
    /\ UNCHANGED decision

Decide(v) ==
    /\ v \notin decision
    /\ \E Q \in Quorum : \A a \in Q : v \in votes[a]
    /\ decision' = decision \cup {v}
    /\ UNCHANGED votes

Next ==
    \/ \E a \in Acceptor, v \in Value : Vote(a, v)
    \/ \E v \in Value : Decide(v)

Spec == Init /\ [][Next]_vars

\* Agreement: at most one value decided
Agreement == Cardinality(decision) <= 1

\* Validity: only proposed values can be decided
Validity == decision \subseteq Value
=============================================================================
```

## Example 8: Two-Phase Commit

```tla
--------------------------- MODULE TwoPhaseCommit ---------------------------
EXTENDS Naturals

CONSTANTS RM  \* Set of resource managers

VARIABLES rmState, tmState, tmPrepared, msgs

vars == <<rmState, tmState, tmPrepared, msgs>>

Message == [type : {"Prepared", "Commit", "Abort"}]

TypeOK ==
    /\ rmState \in [RM -> {"working", "prepared", "committed", "aborted"}]
    /\ tmState \in {"init", "committed", "aborted"}
    /\ tmPrepared \subseteq RM
    /\ msgs \subseteq Message

Init ==
    /\ rmState = [r \in RM |-> "working"]
    /\ tmState = "init"
    /\ tmPrepared = {}
    /\ msgs = {}

\* RM sends Prepared message
RMPrepare(r) ==
    /\ rmState[r] = "working"
    /\ rmState' = [rmState EXCEPT ![r] = "prepared"]
    /\ msgs' = msgs \cup {[type |-> "Prepared"]}
    /\ UNCHANGED <<tmState, tmPrepared>>

\* TM receives Prepared from RM
TMRcvPrepared(r) ==
    /\ tmState = "init"
    /\ [type |-> "Prepared"] \in msgs
    /\ tmPrepared' = tmPrepared \cup {r}
    /\ UNCHANGED <<rmState, tmState, msgs>>

\* TM commits when all RMs prepared
TMCommit ==
    /\ tmState = "init"
    /\ tmPrepared = RM
    /\ tmState' = "committed"
    /\ msgs' = msgs \cup {[type |-> "Commit"]}
    /\ UNCHANGED <<rmState, tmPrepared>>

\* TM aborts
TMAbort ==
    /\ tmState = "init"
    /\ tmState' = "aborted"
    /\ msgs' = msgs \cup {[type |-> "Abort"]}
    /\ UNCHANGED <<rmState, tmPrepared>>

\* RM commits upon receiving Commit
RMRcvCommit(r) ==
    /\ rmState[r] = "prepared"
    /\ [type |-> "Commit"] \in msgs
    /\ rmState' = [rmState EXCEPT ![r] = "committed"]
    /\ UNCHANGED <<tmState, tmPrepared, msgs>>

\* RM aborts upon receiving Abort
RMRcvAbort(r) ==
    /\ rmState[r] \in {"working", "prepared"}
    /\ [type |-> "Abort"] \in msgs
    /\ rmState' = [rmState EXCEPT ![r] = "aborted"]
    /\ UNCHANGED <<tmState, tmPrepared, msgs>>

Next ==
    \/ \E r \in RM : RMPrepare(r)
    \/ \E r \in RM : TMRcvPrepared(r)
    \/ TMCommit
    \/ TMAbort
    \/ \E r \in RM : RMRcvCommit(r)
    \/ \E r \in RM : RMRcvAbort(r)

Spec == Init /\ [][Next]_vars

\* Consistency: all RMs reach same decision
Consistency ==
    /\ \A r1, r2 \in RM : ~(rmState[r1] = "committed" /\ rmState[r2] = "aborted")
    /\ (tmState = "committed") => (\A r \in RM : rmState[r] /= "aborted")
    /\ (tmState = "aborted") => (\A r \in RM : rmState[r] /= "committed")
=============================================================================
```

## Common Specification Patterns Summary

1. **State Machine**: Define states, transitions, and invariants
2. **Concurrent Processes**: Use process IDs as indices into state arrays
3. **Message Passing**: Model channels as sets/sequences of messages
4. **Transactions**: Snapshot isolation with conflict detection
5. **Mutual Exclusion**: Entry/exit protocols with safety properties
6. **Producer-Consumer**: Bounded buffers with blocking
7. **Consensus**: Quorum-based voting
8. **Two-Phase Commit**: Coordinator-participant protocols
