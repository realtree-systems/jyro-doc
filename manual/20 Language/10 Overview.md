# Language Overview

A Jyro script is a sequence of statements that run from the first to the last. There is no entry point and no module system. The script reads the data the host supplies through `Input`, which is read-only, and writes its result to `Output`, which starts as an empty object. The value of `Output` when the script ends is the result of the run.

```jyro
# input: { "items": [ { "name": "bolt", "qty": 4, "price": 0.25 }, { "name": "nut", "qty": 10, "price": 0.1 } ] }
# The total cost of an order, and the names of its lines.

var total = 0
var names = []

foreach item in Input.items do
    total += item.qty * item.price
    names = Append(names, item.name)
end

Output.total = total
Output.names = names
Output.large = total > 100
```

```json
{ "total": 2, "names": ["bolt", "nut"], "large": false }
```

The elements of the language are:

| Element | Description |
| --- | --- |
| Values | null, boolean, number, string, array, object, and function. Numbers are exact decimals where the value fits. |
| Variables | Declared with `var`, block scoped, optionally constrained to a type. |
| Expressions | Arithmetic, string concatenation with `&`, comparison, logical operators, null coalescing, a conditional operator, type checks, property and index access, calls, and lambdas. |
| Statements | `if`, `switch`, `while`, `for`, `foreach`, `break`, `continue`, `return`, `exit`, `fail`, `delete`, and assignment. Blocks close with `end`. |
| Functions | Declared with `func` at the top level. A body sees its parameters, its locals, and `Input`. |
| Unions | Declared with `union`. A variant is constructed by calling its name and taken apart with `match`. |
| Standard library | Functions for strings, arrays, math, dates, objects, validation, queries, and lambdas, available when the host enables them. |

A script is compiled before it runs. Compilation parses the source, validates the static rules, and links every call to a function. Each stage reports its errors as diagnostics with a code and a position. A run that reaches a runtime error stops at that error and reports it the same way. Every run is bounded by limits on execution time, statement count, loop iterations, and call depth.

The host decides which functions a script can call. The standard library is enabled by the host; a host may also add its own functions and attach libraries of `func` and `union` declarations.
