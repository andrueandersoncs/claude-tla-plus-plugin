---
description: Generate invariants for an existing TLA+ specification
---

# Generate TLA+ Invariants

Analyze the TLA+ specification and generate appropriate invariants for: "$ARGUMENTS"

## Instructions

1. **Read the Specification**: Understand the state variables, actions, and intended behavior.

2. **Generate Invariants**:

   **TypeInvariant** (always include):
   - Specify types for all variables
   - Use proper TLA+ type idioms:
     ```tla
     TypeInvariant ==
         /\ var1 \in SomeSet
         /\ var2 \in [Key -> Value]
         /\ var3 \subseteq AllowedValues
         /\ var4 \in Seq(Element)
     ```

   **Safety Invariants**:
   - Mutual exclusion (if applicable)
   - Data integrity constraints
   - Resource bounds (buffer limits, etc.)
   - State consistency requirements

   **Structural Invariants**:
   - Relationships between variables
   - Derived constraints from system requirements

3. **Format**:
   ```tla
   \* Type correctness
   TypeInvariant == ...

   \* Safety property: [description]
   SafetyProperty1 == ...

   \* Invariant: [description]
   Invariant1 == ...
   ```

4. **Update Configuration**: Add invariants to the .cfg file:
   ```cfg
   INVARIANTS
       TypeInvariant
       SafetyProperty1
   ```
