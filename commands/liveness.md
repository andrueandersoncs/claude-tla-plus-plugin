---
description: Generate liveness properties for a TLA+ specification
---

# Generate Liveness Properties

Generate liveness properties (temporal properties) for: "$ARGUMENTS"

## Instructions

Liveness properties assert that "something good eventually happens."

1. **Common Liveness Patterns**:

   **Termination**:
   ```tla
   Termination == <>(\A p \in Procs : pc[p] = "done")
   ```

   **Progress (leads-to)**:
   ```tla
   Progress == \A p \in Procs :
       pc[p] = "waiting" ~> pc[p] = "served"
   ```

   **Eventual Response**:
   ```tla
   EventualResponse == \A req \in Requests :
       req \in pending ~> req \in completed
   ```

   **Starvation Freedom**:
   ```tla
   StarvationFree == \A p \in Procs :
       pc[p] = "trying" ~> pc[p] = "critical"
   ```

   **Eventual Consistency**:
   ```tla
   EventualConsistency ==
       <>(\A r1, r2 \in Replicas : data[r1] = data[r2])
   ```

2. **Fairness Requirements**:
   - Liveness often requires fairness assumptions
   - Weak fairness (WF): If continuously enabled, eventually happens
   - Strong fairness (SF): If repeatedly enabled, eventually happens

   ```tla
   FairSpec == Spec
       /\ \A p \in Procs : WF_vars(Action(p))
       /\ SF_vars(CriticalAction)
   ```

3. **Generate**:
   - Appropriate liveness properties
   - Required fairness conditions
   - Update to .cfg file:
     ```cfg
     SPECIFICATION FairSpec
     PROPERTIES
         Termination
         Progress
     ```

4. **Note**: Liveness checking is more expensive than safety checking. Test with small models first.
