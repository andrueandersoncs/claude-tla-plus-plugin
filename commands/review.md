---
description: Review and improve an existing TLA+ specification
---

# Review TLA+ Specification

Review the TLA+ specification and provide feedback on: "$ARGUMENTS"

## Review Checklist

1. **Structure and Style**:
   - Proper module header with documentation
   - Clear separation of constants, variables, operators
   - Consistent naming conventions
   - Adequate comments explaining intent

2. **Correctness**:
   - Init covers all variables
   - Next is complete (all possible transitions)
   - Actions have proper preconditions
   - UNCHANGED clauses are correct
   - No unintended variable shadowing

3. **Type Safety**:
   - TypeInvariant is defined
   - All variables have clear types
   - Set comprehensions are well-formed
   - Function domains are explicit

4. **Properties**:
   - Safety properties are clearly stated
   - Liveness properties (if any) have fairness
   - Properties match system requirements

5. **Model Checking**:
   - Constants are appropriately bounded
   - Symmetry is used where applicable
   - State constraints limit explosion

6. **Common Issues to Check**:
   - Deadlock possibility
   - Missing UNCHANGED clauses
   - Overly restrictive preconditions
   - Unbounded state growth
   - Missing fairness for liveness

7. **Output**:
   - Summary of findings
   - Specific recommendations with code examples
   - Priority: Critical > Important > Minor
