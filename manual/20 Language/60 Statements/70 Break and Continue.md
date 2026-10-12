# Break and Continue

`break` ends the innermost enclosing `while`, `for`, or `foreach` loop. `continue` ends the current iteration of the innermost enclosing loop and proceeds to the next: the next evaluation of a `while` condition, the next value of a `for` sequence, or the next element of a `foreach` collection.

```jyro
var kept = []
for i in 0 to 10 do
    if i % 2 == 0 then
        continue
    end
    if i > 6 then
        break
    end
    kept = Append(kept, i)
end
Output.kept = kept
```

```json
{ "kept": [1, 3, 5] }
```

Both statements apply to the innermost loop only. A `switch` or `match` inside a loop does not change which loop is affected.

Either statement outside a loop is refused when the script is compiled.

```jyro
break
```

```text
[JM3400] Ln 1, Col 1: 'break' statement outside of loop
```

A `break` or `continue` inside a function body applies to a loop inside that function. A function cannot affect a loop in its caller.
