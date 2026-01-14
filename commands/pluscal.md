---
description: Generate a PlusCal algorithm with TLA+ translation
---

# Generate PlusCal Algorithm

Generate a PlusCal algorithm for: "$ARGUMENTS"

## Instructions

PlusCal is an algorithmic language that translates to TLA+. Use it when the system is naturally expressed as sequential/concurrent processes.

1. **Structure the Algorithm**:
```tla
(*--algorithm AlgorithmName
variables globalVar = initialValue;

define
  \* Helper operators
end define;

process ProcName \in ProcessSet
variables localVar = value;
begin
  label1:
    statement;
  label2:
    while condition do
      statement;
    end while;
end process;

end algorithm; *)
```

2. **PlusCal Constructs to Use**:
   - `either/or` for nondeterministic choice
   - `with x \in Set` for nondeterministic selection
   - `await condition` for blocking
   - `fair process` for weak fairness
   - `fair+ process` for strong fairness

3. **Generate**:
   - Complete PlusCal algorithm with comments
   - The TLA+ translation will be added by the PlusCal translator
   - Invariants and properties to check
   - TLC configuration file

4. **Note**: After generating, run the PlusCal translator:
   - In TLA+ Toolbox: File → Translate PlusCal Algorithm
   - Command line: `java pcal.trans Spec.tla`
