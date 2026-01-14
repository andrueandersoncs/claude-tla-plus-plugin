# TLC Model Checker Configuration Guide

TLC (Temporal Logic Checker) is the model checker for TLA+ specifications. This guide covers configuration files and best practices.

## Configuration File Format (.cfg)

Create a `.cfg` file with the same base name as your `.tla` file:

```cfg
\* Comments start with \*

\* Specify the specification formula
SPECIFICATION Spec

\* Alternatively, for Init/Next style:
INIT Init
NEXT Next

\* Constants - using literal values
CONSTANTS
    NumProcesses = 3
    MaxValue = 10
    BufferSize = 5

\* Constants - using model values (uninterpreted)
CONSTANTS
    Procs = {p1, p2, p3}
    Nodes = {n1, n2, n3, n4}

\* Constants - using sets
CONSTANTS
    Values = {v1, v2, v3}
    Keys = {k1, k2}

\* Invariants to check (safety properties)
INVARIANTS
    TypeInvariant
    Safety
    MutualExclusion

\* Properties to check (including liveness)
PROPERTIES
    Liveness
    Termination
    Progress

\* Symmetry sets for optimization
SYMMETRY
    Permutations(Procs)

\* State constraint to limit search
CONSTRAINT
    StateConstraint

\* Action constraint
ACTION_CONSTRAINT
    ActionConstraint

\* Check deadlock (default: true)
CHECK_DEADLOCK TRUE

\* Alias for trace exploration
ALIAS
    Alias
```

## Complete Configuration Examples

### Example 1: Key-Value Store Configuration

```cfg
\* MCKeyValueStore.cfg
SPECIFICATION Spec

CONSTANTS
    Key = {k1, k2}
    Val = {v1, v2}
    TxId = {t1, t2}

INVARIANTS
    TypeInvariant
    TxLifecycle

SYMMETRY
    Permutations(Key) \union Permutations(Val) \union Permutations(TxId)
```

### Example 2: Bakery Algorithm Configuration

```cfg
\* MCBakery.cfg
SPECIFICATION Spec

CONSTANTS
    N = 3

INVARIANTS
    TypeOK
    MutualExclusion

PROPERTIES
    StarvationFree

CHECK_DEADLOCK TRUE
```

### Example 3: Elevator System Configuration

```cfg
\* ElevatorSafety.cfg
SPECIFICATION Spec

CONSTANTS
    Person = {person1, person2}
    Elevator = {e1}
    FloorCount = 3

INVARIANTS
    TypeInvariant
    SafetyInvariant

\* Don't check liveness for safety verification
\* PROPERTIES
\*     TemporalInvariant

SYMMETRY
    Permutations(Person)

CHECK_DEADLOCK FALSE
```

### Example 4: Consensus Protocol Configuration

```cfg
\* MCConsensus.cfg
SPECIFICATION Spec

CONSTANTS
    Value = {v1, v2}
    Acceptor = {a1, a2, a3}
    Quorum = {{a1, a2}, {a1, a3}, {a2, a3}}

INVARIANTS
    TypeOK
    Agreement
    Validity

SYMMETRY
    Permutations(Value) \union Permutations(Acceptor)
```

### Example 5: Two-Phase Commit Configuration

```cfg
\* MC2PC.cfg
SPECIFICATION Spec

CONSTANTS
    RM = {rm1, rm2, rm3}

INVARIANTS
    TypeOK
    Consistency

SYMMETRY
    Permutations(RM)

CHECK_DEADLOCK TRUE
```

### Example 6: Producer-Consumer Configuration

```cfg
\* MCProducerConsumer.cfg
SPECIFICATION FairSpec

CONSTANTS
    Producers = {prod1, prod2}
    Consumers = {cons1}
    BufferSize = 3
    Data = {d1, d2}

INVARIANTS
    TypeInvariant
    BufferSafety

PROPERTIES
    Progress

SYMMETRY
    Permutations(Data)

\* Limit state space for testing
CONSTRAINT
    Len(buffer) <= BufferSize
```

## Configuration Options Explained

### SPECIFICATION vs INIT/NEXT

```cfg
\* Use SPECIFICATION for temporal formulas with fairness
SPECIFICATION Spec
\* Where Spec == Init /\ [][Next]_vars /\ Fairness

\* Use INIT/NEXT for simple safety checking
INIT Init
NEXT Next
```

### Model Values vs Ordinary Values

**Model Values** (uninterpreted constants):
```cfg
CONSTANTS
    Procs = {p1, p2, p3}  \* Model values
```
- TLC creates fresh symbolic values
- Good for abstract identifiers
- Enable symmetry reduction

**Ordinary Values** (concrete values):
```cfg
CONSTANTS
    N = 3
    MaxItems = 10
```
- Use Naturals, Integers, etc.
- Good for numeric bounds

### Symmetry Optimization

Symmetry reduces state space by treating permutations as equivalent:

```cfg
SYMMETRY
    Permutations(Procs)

\* Multiple symmetry sets
SYMMETRY
    Permutations(Procs) \union Permutations(Values)
```

**Requirements for symmetry:**
- Constants must be model values
- Specification must be symmetric (treating all elements equivalently)

### State Constraints

Limit exploration to states satisfying constraint:

```cfg
CONSTRAINT
    counter < 100 /\ Len(buffer) <= 10

\* Multiple constraints (conjuncted)
CONSTRAINT StateConstraint1
CONSTRAINT StateConstraint2
```

### Action Constraints

Limit which actions TLC explores:

```cfg
ACTION_CONSTRAINT
    \* Only explore actions where counter increases by at most 1
    counter' <= counter + 1
```

## TLC Command Line Options

Run TLC from command line:

```bash
# Basic run
java -jar tla2tools.jar -config MCSpec.cfg Spec.tla

# With workers for parallelism
java -jar tla2tools.jar -workers 4 -config MCSpec.cfg Spec.tla

# Simulation mode (random exploration)
java -jar tla2tools.jar -simulate -depth 100 Spec.tla

# Generate trace
java -jar tla2tools.jar -dump dot,colorize states.dot Spec.tla

# Check specific properties
java -jar tla2tools.jar -config MCSpec.cfg \
    -invariant TypeOK \
    -property Liveness \
    Spec.tla
```

## Common TLC Options

| Option | Description |
|--------|-------------|
| `-workers N` | Use N worker threads |
| `-simulate` | Random simulation mode |
| `-depth N` | Maximum trace depth |
| `-checkpoint M` | Checkpoint every M minutes |
| `-recover path` | Recover from checkpoint |
| `-deadlock` | Check for deadlock |
| `-dump fmt file` | Dump state graph |
| `-coverage M` | Report coverage every M minutes |
| `-debugger` | Enable debugger |

## Best Practices

### 1. Start Small
```cfg
\* Start with minimal constants
CONSTANTS
    N = 2
    MaxValue = 3

\* Gradually increase after verification
```

### 2. Use Symmetry When Possible
```cfg
\* Good: symmetric model values
CONSTANTS Procs = {p1, p2, p3}
SYMMETRY Permutations(Procs)

\* Bad: asymmetric (no symmetry possible)
CONSTANTS Procs = {1, 2, 3}
```

### 3. Add State Constraints for Large Models
```cfg
\* Limit exploration depth
CONSTRAINT
    clock < 10 /\
    \A p \in Procs : counter[p] < 5
```

### 4. Separate Safety and Liveness Checking
```cfg
\* SafetyCheck.cfg
SPECIFICATION Spec  \* Without fairness
INVARIANTS TypeOK Safety

\* LivenessCheck.cfg
SPECIFICATION FairSpec  \* With fairness
PROPERTIES Liveness
```

### 5. Use ALIAS for Debugging
```cfg
\* Define readable state representation
ALIAS
    [
        state |-> state,
        pending |-> Len(queue),
        active |-> {p \in Procs : pc[p] = "active"}
    ]
```

## Troubleshooting

### "Attempted to compute CHOOSE..." Error
- Ensure sets are non-empty before CHOOSE
- Add preconditions to actions

### State Space Explosion
- Reduce constant values
- Add state constraints
- Use symmetry reduction
- Try simulation mode first

### Liveness Checking is Slow
- Check safety properties first
- Use smaller models for liveness
- Consider fairness requirements carefully

### "Invariant violated" with no trace
- Enable `-dump` to see states
- Add TypeInvariant first
- Check Init predicate
