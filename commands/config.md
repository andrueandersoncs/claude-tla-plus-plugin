---
description: Generate or update a TLC configuration file for a TLA+ spec
---

# Generate TLC Configuration

Generate a TLC model checker configuration for: "$ARGUMENTS"

## Instructions

1. **Analyze the Specification**:
   - Identify all CONSTANTS
   - Identify all defined invariants and properties
   - Determine if symmetry is applicable

2. **Generate Configuration File**:

   ```cfg
   \* Configuration for ModuleName

   SPECIFICATION Spec

   \* Constants
   CONSTANTS
       ConstantName = value
       SetConstant = {v1, v2, v3}

   \* Invariants (safety)
   INVARIANTS
       TypeInvariant
       SafetyProperty

   \* Properties (liveness) - optional
   PROPERTIES
       LivenessProperty

   \* Symmetry optimization
   SYMMETRY
       Permutations(SetConstant)

   \* State constraint (optional)
   CONSTRAINT
       StateConstraint

   CHECK_DEADLOCK TRUE
   ```

3. **Recommendations**:
   - Start with small constant values
   - Add symmetry for model value sets
   - Use constraints for large state spaces
   - Separate safety and liveness checking

4. **Multiple Configurations**:
   Create separate configs for different purposes:
   - `MC_small.cfg` - Quick sanity check
   - `MC_safety.cfg` - Full safety verification
   - `MC_liveness.cfg` - Liveness with fairness

5. **Output**: Complete .cfg file with explanatory comments
