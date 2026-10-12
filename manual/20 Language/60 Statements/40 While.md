# While

```jyro
# fragment
while condition do
    statements
end
```

The condition is evaluated for truthiness before each iteration. The block runs while the condition is truthy. A loop whose condition is falsy at the start runs zero times.

```jyro
var n = 1
var steps = 0
while n < 100 do
    n *= 2
    steps++
end
Output.n = n
Output.steps = steps
```

```json
{ "n": 128, "steps": 7 }
```

`break` ends the loop and `continue` starts the next evaluation of the condition; see [Break and Continue](/docs/language/statements/break-and-continue).

Each iteration counts toward the run's loop iteration limit, which is shared by every loop in the run. A loop that does not end is stopped by that limit or by the execution time limit; see [Resource Limits](/docs/language/errors-and-limits/resource-limits).
