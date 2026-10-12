# Assignment

An assignment is a statement. It stores a value in a variable, a property, or an array element.

```jyro
# fragment
target = expression
target: type = expression
target op= expression
```

Assignment is not an expression: `a = b = 1` is a parse error, and an assignment cannot appear inside a condition or an argument.

## Targets

A target is an identifier, a property access, or an index access. A parenthesized expression and any other expression are refused with JM3200 `InvalidAssignmentTarget`. An identifier must name a declared variable.

```jyro
var a = 1
var o = { list: [0, 0] }
a = 2
o.list[1] = a
o["name"] = "x"
Output.o = o
```

```json
{ "o": { "list": [0, 2], "name": "x" } }
```

```jyro
var a = 1
(a) = 2
```

```text
[JM3200] Ln 2, Col 1: Invalid assignment target
```

The rules for writing a property or an element are given in [Objects](/docs/language/types-and-values/objects) and [Arrays](/docs/language/types-and-values/arrays).

## Compound assignment

A compound assignment applies an operator to the current value of the target and the right-hand value, and stores the result. The operators are `+=`, `-=`, `*=`, `/=`, `%=`, and `&=`. The target, including any index expression, is evaluated once.

```jyro
var n = 10
n += 5
n -= 3
n *= 2
n /= 4
n %= 4
var s = "ab"
s &= "c"
Output.n = n
Output.s = s
```

```json
{ "n": 2, "s": "abc" }
```

```jyro
var calls = { n: 0 }
func Next(c)
    c.n += 1
    return c.n
end
var a = [10, 20, 30]
a[Next(calls) - 1] += 5
Output.a = a
Output.calls = calls.n
```

```json
{ "a": [15, 20, 30], "calls": 1 }
```

## Type hints on assignment

A type may be written on an assignment with `=` to a property or an element, or on the first assignment to a variable that was declared without a type. The value is converted to that type.

```jyro
Output.count: number = "5"
Output.flag: boolean = 1
```

```json
{ "count": 5, "flag": true }
```

A type hint is refused on a compound assignment with JM3202 `TypeHintOnCompoundAssignment`, and on a target that already has a value with JM3203 `DuplicateTypeHint`.

```jyro
Output.x: number += 1
```

```text
[JM3202] Ln 1, Col 1: Type hint is not allowed on a compound assignment
```

```jyro
var n = 1
n: number = 2
```

```text
[JM3203] Ln 2, Col 1: Type hint on 'n' must appear on its first assignment
```
