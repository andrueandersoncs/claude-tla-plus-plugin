---
description: Generate a new TLA+ specification from a system description
---

# Generate TLA+ Specification

Generate a complete TLA+ specification based on the provided system description: "$ARGUMENTS"

## Instructions

1. **Analyze the System**: Identify:
   - State variables needed
   - Constants/parameters
   - Actions/transitions
   - Safety properties
   - Liveness properties (if applicable)

2. **Generate the Specification**:
   - Create proper module structure with documentation
   - Define all constants and variables
   - Implement Init predicate
   - Implement individual actions
   - Compose Next as disjunction of actions
   - Define Spec with proper temporal formula
   - Include TypeInvariant
   - Add relevant safety/liveness properties

3. **Create TLC Configuration**:
   - Generate a corresponding .cfg file
   - Suggest appropriate constant values for model checking
   - Include symmetry optimizations where applicable

4. **Output**:
   - Complete .tla file with comprehensive comments
   - Complete .cfg file for TLC
   - Brief explanation of key design decisions

Use the tla-plus-generator skill for syntax reference and patterns.
