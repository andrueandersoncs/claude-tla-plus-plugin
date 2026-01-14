---
name: tla-plus-generator
description: Generate TLA+ specifications, PlusCal algorithms, and TLC model configurations for formal verification. Use when the user wants to formally specify systems, verify concurrent algorithms, model distributed systems, check safety/liveness properties, or create state machine specifications.
---

# TLA+ Specification Generator

You are an expert in TLA+ (Temporal Logic of Actions), a formal specification language for describing and verifying concurrent and distributed systems. When generating TLA+ specifications, follow these guidelines and patterns.

## Core Concepts

### Module Structure
Every TLA+ specification follows this structure:
```tla
--------------------------- MODULE ModuleName ---------------------------
(* Module documentation *)
EXTENDS Naturals, Sequences, FiniteSets, TLC  \* Standard modules

CONSTANTS ConstantName  \* Declared constants
VARIABLES var1, var2    \* State variables
----------------------------------------------------------------------------

\* Definitions, operators, and actions go here

=============================================================================
```

### Variable Declaration Patterns
- Use meaningful names that describe the state
- Group related variables together
- Document the type/domain of each variable

### Init and Next State Relations
```tla
Init == \* Initial state predicate
    /\ var1 = initialValue1
    /\ var2 = initialValue2

Next == \* Next-state relation (disjunction of actions)
    \/ Action1
    \/ Action2
    \/ \E x \in SomeSet : Action3(x)

Spec == Init /\ [][Next]_<<var1, var2>>  \* Full specification
```

## TLA+ Syntax Reference

### Logical Operators
- `/\` - conjunction (AND)
- `\/` - disjunction (OR)
- `~` - negation (NOT)
- `=>` - implication
- `<=>` - equivalence
- `TRUE`, `FALSE` - boolean constants

### Set Operations
- `\in` - element of
- `\notin` - not element of
- `\cup` - union
- `\cap` - intersection
- `\subseteq` - subset or equal
- `SUBSET S` - powerset of S
- `UNION S` - union of all sets in S
- `{x \in S : P(x)}` - set filter
- `{f(x) : x \in S}` - set map
- `Cardinality(S)` - size of finite set (requires FiniteSets)

### Function Operations
- `[x \in S |-> expr]` - function definition
- `f[x]` - function application
- `DOMAIN f` - domain of function
- `[f EXCEPT ![x] = y]` - function update
- `[f EXCEPT ![x] = @ + 1]` - update using current value (@)
- `[S -> T]` - set of all functions from S to T

### Temporal Operators
- `[]P` - always P (invariant)
- `<>P` - eventually P
- `P ~> Q` - P leads to Q
- `[][A]_v` - A or stuttering step
- `WF_v(A)` - weak fairness
- `SF_v(A)` - strong fairness

### CHOOSE Operator
```tla
CHOOSE x \in S : P(x)  \* Select arbitrary element satisfying P
```

### LET-IN Expressions
```tla
LET
  localVar == expression
  helper(x) == anotherExpression
IN
  resultExpression
```

## Common Patterns

### Type Invariant Pattern
Always define a TypeInvariant to catch specification bugs early:
```tla
TypeInvariant ==
    /\ var1 \in SomeSet
    /\ var2 \in [Key -> Value]
    /\ var3 \subseteq AllowedValues
```

### State Machine Pattern
For modeling state machines:
```tla
CONSTANTS States, InitialState
VARIABLES state, data

Init ==
    /\ state = InitialState
    /\ data = initialData

Transition(from, to) ==
    /\ state = from
    /\ state' = to
    /\ SomeCondition
    /\ data' = UpdatedData

Next ==
    \/ Transition("idle", "running")
    \/ Transition("running", "completed")
    \/ Transition("running", "failed")
```

### Concurrent Process Pattern
For multiple concurrent processes:
```tla
CONSTANTS Procs  \* Set of process IDs
VARIABLES pc, localState

Init ==
    /\ pc = [p \in Procs |-> "start"]
    /\ localState = [p \in Procs |-> InitialValue]

Step(p) ==
    /\ pc[p] = "start"
    /\ pc' = [pc EXCEPT ![p] = "next"]
    /\ localState' = [localState EXCEPT ![p] = NewValue]

Next == \E p \in Procs : Step(p)
```

### Message Passing Pattern
For distributed systems with message passing:
```tla
VARIABLES messages  \* Set or bag of messages

Send(m) ==
    /\ messages' = messages \cup {m}

Receive(m) ==
    /\ m \in messages
    /\ messages' = messages \ {m}
```

### Transaction Pattern
For systems with transactions:
```tla
VARIABLES store, tx, snapshotStore

OpenTx(t) ==
    /\ t \notin tx
    /\ tx' = tx \cup {t}
    /\ snapshotStore' = [snapshotStore EXCEPT ![t] = store]
    /\ UNCHANGED store

CommitTx(t) ==
    /\ t \in tx
    /\ NoConflicts(t)  \* Check for conflicts
    /\ store' = MergeChanges(store, snapshotStore[t])
    /\ tx' = tx \ {t}
```

## Safety and Liveness Properties

### Safety Properties (Nothing bad happens)
```tla
\* Mutual exclusion
MutualExclusion == \A i, j \in Procs :
    (i /= j) => ~(pc[i] = "cs" /\ pc[j] = "cs")

\* No deadlock
NoDeadlock == ENABLED Next

\* Data integrity
DataIntegrity == \A k \in DOMAIN store : store[k] \in ValidValues
```

### Liveness Properties (Something good eventually happens)
```tla
\* Progress
Progress == \A p \in Procs : pc[p] = "waiting" ~> pc[p] = "done"

\* Termination
Termination == <>(\A p \in Procs : pc[p] = "done")

\* Eventual consistency
EventualConsistency == <>(\A r \in Replicas : data[r] = data[CHOOSE x \in Replicas : TRUE])
```

### Fairness Conditions
```tla
\* Weak fairness: if action is continuously enabled, it eventually occurs
FairSpec == Spec /\ WF_vars(SomeAction)

\* Strong fairness: if action is repeatedly enabled, it eventually occurs
StrongFairSpec == Spec /\ SF_vars(SomeAction)
```

## TLC Model Configuration

Create a `.cfg` file to configure TLC model checker:

```cfg
\* Specify constants
CONSTANTS
    NumProcesses = 3
    MaxValue = 10

\* Or use model values for uninterpreted sets
CONSTANTS
    Procs = {p1, p2, p3}

\* Specify the specification to check
SPECIFICATION Spec

\* Invariants to check (safety properties)
INVARIANTS
    TypeInvariant
    MutualExclusion
    NoDeadlock

\* Properties to check (can include liveness)
PROPERTIES
    Termination

\* Symmetry optimization (if applicable)
SYMMETRY Permutations
```

## Best Practices

1. **Start Simple**: Begin with a minimal specification and add complexity incrementally
2. **Type Invariants First**: Define TypeInvariant before other properties
3. **Use ASSUMES**: Document assumptions about constants
4. **Comment Extensively**: TLA+ specs serve as documentation
5. **Small Models First**: Test with small constant values before scaling
6. **Check for Deadlock**: Ensure the system can always make progress
7. **Separate Concerns**: Use separate modules for complex specifications

## Additional Resources

For detailed syntax reference, see [syntax-reference.md](syntax-reference.md)
For more example patterns, see [patterns-examples.md](patterns-examples.md)
For TLC configuration details, see [tlc-configuration.md](tlc-configuration.md)

## When Generating Specifications

1. Ask clarifying questions about:
   - What system is being modeled?
   - What are the key state variables?
   - What actions/transitions are possible?
   - What properties should hold (safety/liveness)?
   - What are the constants/parameters?

2. Structure the specification:
   - Start with module header and extends
   - Declare constants and variables
   - Define helper operators
   - Define Init predicate
   - Define individual actions
   - Define Next as disjunction of actions
   - Define Spec with temporal formula
   - Define invariants and properties

3. Create corresponding .cfg file for TLC model checking

4. Suggest appropriate constant values for model checking
