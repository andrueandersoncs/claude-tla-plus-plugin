# TLA+ Complete Syntax Reference

## Module Structure

```tla
---- MODULE ModuleName ----
(* Multi-line comments use (* *) *)
\* Single-line comments start with \*

EXTENDS Module1, Module2   \* Import standard modules
CONSTANTS Const1, Const2   \* Declare constants
VARIABLES var1, var2       \* Declare variables
ASSUME Const1 \in Nat      \* Assumptions about constants

\* Definitions and operators

====
```

## Standard Modules

### Naturals
```tla
EXTENDS Naturals
\* Provides: Nat, +, -, *, ^, <, >, <=, >=, %, \div
\* Nat = {0, 1, 2, ...}
\* a..b = {a, a+1, ..., b}  (integer range)
```

### Integers
```tla
EXTENDS Integers
\* Provides: Int, -a (negation)
\* Int = {..., -2, -1, 0, 1, 2, ...}
```

### Reals
```tla
EXTENDS Reals
\* Provides: Real, /, Infinity
```

### Sequences
```tla
EXTENDS Sequences
\* Seq(S) - set of all finite sequences with elements from S
\* Head(s) - first element
\* Tail(s) - all but first element
\* Append(s, e) - append element to sequence
\* s \o t - concatenation
\* Len(s) - length
\* s[i] - element at index i (1-based)
\* SubSeq(s, m, n) - subsequence from m to n
\* SelectSeq(s, Test(_)) - filter sequence
```

### FiniteSets
```tla
EXTENDS FiniteSets
\* IsFiniteSet(S) - TRUE if S is finite
\* Cardinality(S) - number of elements in S
```

### Bags (Multisets)
```tla
EXTENDS Bags
\* EmptyBag - empty bag
\* IsABag(B) - TRUE if B is a bag
\* BagIn(e, B) - TRUE if e is in bag B
\* BagToSet(B) - convert bag to set
\* SetToBag(S) - convert set to bag
\* BagUnion(B1, B2) - bag union
\* CopiesIn(e, B) - count of e in B
```

### TLC
```tla
EXTENDS TLC
\* Print(val, out) - print val, return out
\* PrintT(val) - print val, return TRUE
\* Assert(cond, msg) - assert condition
\* JavaTime - current time in ms
\* Permutations(S) - symmetry set for TLC
\* SortSeq(s, Op(_, _)) - sort sequence
```

## Operators and Expressions

### Boolean Operators
```tla
TRUE                    \* Boolean true
FALSE                   \* Boolean false
~P                      \* Negation (NOT)
P /\ Q                  \* Conjunction (AND)
P \/ Q                  \* Disjunction (OR)
P => Q                  \* Implication
P <=> Q                 \* Equivalence (iff)
P # Q                   \* Not equal (same as /=)
IF P THEN e1 ELSE e2    \* Conditional expression
CASE p1 -> e1           \* Case expression
  [] p2 -> e2
  [] OTHER -> e3
```

### Set Operators
```tla
{e1, e2, e3}            \* Set enumeration
{x \in S : P(x)}        \* Set filter (comprehension)
{f(x) : x \in S}        \* Set map
x \in S                 \* Membership
x \notin S              \* Non-membership
S \cup T                \* Union
S \cap T                \* Intersection
S \ T                   \* Set difference
S \subseteq T           \* Subset or equal
SUBSET S                \* Powerset
UNION S                 \* Distributed union
S \X T                  \* Cartesian product
```

### Function Operators
```tla
[x \in S |-> e]         \* Function definition
f[x]                    \* Function application
DOMAIN f                \* Domain of function
[S -> T]                \* Set of functions from S to T
[f EXCEPT ![a] = b]     \* Function update
[f EXCEPT ![a] = @ + 1] \* Update using current value
[f EXCEPT ![a][b] = c]  \* Nested update
[f EXCEPT ![a] = b,     \* Multiple updates
          ![c] = d]
```

### Record Operators
```tla
[field1 |-> v1, field2 |-> v2]    \* Record construction
r.field                            \* Field access
[field1 : S1, field2 : S2]        \* Set of records
[r EXCEPT !.field = v]            \* Record update
```

### Tuple Operators
```tla
<<e1, e2, e3>>          \* Tuple construction
t[i]                    \* Element access (1-based)
S \X T \X U             \* Set of tuples
```

### Quantifiers
```tla
\A x \in S : P(x)       \* Universal quantifier (for all)
\E x \in S : P(x)       \* Existential quantifier (exists)
\A x, y \in S : P(x,y)  \* Multiple variables
\E x \in S, y \in T : P \* Different sets
```

### CHOOSE Operator
```tla
CHOOSE x \in S : P(x)   \* Choose arbitrary element satisfying P
\* Returns unspecified value if no such element exists
\* Deterministic: always returns same value for same inputs

\* Common pattern for unique element:
CHOOSE x \in S : \A y \in S : P(y) => y = x
```

### LET-IN Expressions
```tla
LET
  x == expr1
  f(a) == expr2
  g(a, b) == expr3
IN
  finalExpr
```

### Lambda Expressions
```tla
LAMBDA x : expr         \* Anonymous function
\* Used with higher-order operators like SelectSeq
```

## Temporal Logic Operators

### State Formulas vs Actions
```tla
\* State formula: predicate on state variables
P == x > 0 /\ y \in S

\* Action: relates current and next state
A == x' = x + 1 /\ y' = y

\* Primed variables refer to next state
x'  \* Value of x in next state
```

### Temporal Operators
```tla
[]P                     \* Always P (box)
<>P                     \* Eventually P (diamond)
P ~> Q                  \* P leads to Q (P => <>Q under fairness)
[][A]_v                 \* Always A or stuttering (v unchanged)
<><<A>>_v               \* Eventually A with v change

\* Fairness
WF_v(A)                 \* Weak fairness of A
SF_v(A)                 \* Strong fairness of A
```

### Specification Pattern
```tla
vars == <<x, y, z>>     \* Tuple of all variables

Init == ...             \* Initial state predicate
Next == ...             \* Next-state relation

Spec == Init /\ [][Next]_vars
\* Init holds initially, then always Next or stutter

FairSpec == Spec /\ WF_vars(Next)
\* Add fairness: if enabled, eventually happens
```

## Actions and State Changes

### Action Composition
```tla
\* Conjunction of actions
A1 /\ A2

\* Disjunction of actions
A1 \/ A2

\* Actions with parameters
Action(param) ==
    /\ precondition
    /\ x' = f(param)
    /\ UNCHANGED <<y, z>>

\* UNCHANGED macro
UNCHANGED <<x, y>>      \* Same as x' = x /\ y' = y
UNCHANGED x             \* Same as x' = x
```

### ENABLED Operator
```tla
ENABLED A               \* TRUE if action A is enabled
\* Useful for checking deadlock freedom
NoDeadlock == ENABLED Next
```

## Operator Definitions

### Constant Operators
```tla
Op == expr                      \* Parameterless operator
Op(a) == expr                   \* Unary operator
Op(a, b) == expr                \* Binary operator
Op(a, b, c) == expr             \* Ternary operator

\* Recursive operator (requires RECURSIVE declaration)
RECURSIVE Fact(_)
Fact(n) == IF n = 0 THEN 1 ELSE n * Fact(n-1)
```

### Higher-Order Operators
```tla
\* Operator that takes operator as parameter
Apply(Op(_), x) == Op(x)
Map(Op(_), S) == {Op(x) : x \in S}
```

### Local Definitions
```tla
Op(x) ==
  LET
    helper == x + 1
    f(y) == y * 2
  IN
    f(helper)
```

## Module System

### EXTENDS
```tla
EXTENDS Module1, Module2
\* Imports all definitions from modules
```

### INSTANCE
```tla
INSTANCE Module WITH const1 <- expr1, const2 <- expr2
\* Import with constant substitution

M == INSTANCE Module WITH ...
\* Prefixed import: use as M!Op
```

### LOCAL
```tla
LOCAL Op == expr
\* Definition not exported from module
```

### THEOREM and PROOF
```tla
THEOREM Spec => []Safety
\* Declare theorem (for documentation or proof)

THEOREM TypeCorrect == Spec => []TypeInvariant
<1>1. Init => TypeInvariant
<1>2. TypeInvariant /\ [Next]_vars => TypeInvariant'
<1>. QED BY <1>1, <1>2, PTL DEF Spec
```

## PlusCal (Algorithmic Language)

PlusCal is translated to TLA+ for model checking.

### Basic Structure
```tla
(*--algorithm AlgorithmName
variables x = 0, y \in 1..10;

define
  \* Operator definitions visible in algorithm
  Max(a, b) == IF a > b THEN a ELSE b
end define;

process ProcName \in ProcSet
variables localVar = 0;
begin
  label1:
    x := x + 1;
  label2:
    while x < 10 do
      x := x + 1;
    end while;
end process;

end algorithm; *)
```

### PlusCal Constructs
```tla
\* Assignment
x := expr;

\* Multiple assignment
x := e1 || y := e2;

\* Conditional
if cond then
  stmt1;
elsif cond2 then
  stmt2;
else
  stmt3;
end if;

\* While loop
while cond do
  stmt;
end while;

\* Either-or (nondeterministic choice)
either
  stmt1;
or
  stmt2;
end either;

\* With (nondeterministic selection)
with v \in Set do
  stmt using v;
end with;

\* Await (blocking)
await condition;

\* Assert
assert condition;

\* Print
print expr;

\* Skip
skip;

\* Goto
goto labelName;

\* Call macro
call MacroName(args);

\* Return
return;

\* Procedure definition
procedure ProcName(param)
variables localVar;
begin
  ...
end procedure;
```

### Fairness in PlusCal
```tla
\* Weak fairness (default)
fair process P \in S

\* Strong fairness
fair+ process P \in S

\* Per-label fairness
label:+ stmt;  \* Strong fairness for this label
label:- stmt;  \* No fairness for this label
```
