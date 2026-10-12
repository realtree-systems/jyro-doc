# Logical Operators

| Operator | Result |
| --- | --- |
| `a and b` | `a` when `a` is falsy; otherwise `b` |
| `a or b` | `a` when `a` is truthy; otherwise `b` |
| `not a` | `true` when `a` is falsy; otherwise `false` |

The operands are evaluated for truthiness as described in [Booleans and Null](/docs/language/types-and-values/booleans-and-null). `and` and `or` return one of their operands, not a boolean. `not` always returns a boolean.

```jyro
Output.a = "hello" and "world"
Output.b = 0 and 1
Output.c = null or "fallback"
Output.d = "first" or "second"
Output.e = not ""
Output.f = not [1]
```

```json
{ "a": "world", "b": 0, "c": "fallback", "d": "first", "e": true, "f": false }
```

## Short-circuit evaluation

The right operand of `and` is evaluated only when the left operand is truthy. The right operand of `or` is evaluated only when the left operand is falsy.

```jyro
Output.a = false and 1 / 0
Output.b = true or 1 / 0
```

```json
{ "a": false, "b": true }
```

## Precedence

`not` binds more tightly than `and`, which binds more tightly than `or`. All three bind less tightly than the comparison operators, so `not a == b` is read as `not (a == b)`. See [Operator Precedence](/docs/language/expressions/operator-precedence).
