# Arithmetic Operators

| Operator | Operation | Operands |
| --- | --- | --- |
| `+` | Addition | number, number |
| `-` | Subtraction | number, number |
| `*` | Multiplication | number, number |
| `/` | Division | number, number |
| `%` | Remainder, with the sign of the dividend | number, number |
| `-` (unary) | Negation | number |

Operands are not converted. An operand of another type is a runtime error.

```jyro
Output.a = 7 + 2
Output.b = 7 - 2
Output.c = 7 * 2
Output.d = 7 / 2
Output.e = 7 % 2
Output.f = -(7 % 2)
```

```json
{ "a": 9, "b": 5, "c": 14, "d": 3.5, "e": 1, "f": -1 }
```

The arithmetic of exact and inexact numbers, division by zero, and overflow are described in [Numbers](/docs/language/types-and-values/numbers).

## Errors

An operand that is not a number is refused with JM5206 `UnsupportedBinaryOperation`, or JM5207 `UnsupportedUnaryOperation` for unary minus.

```jyro
var s = "a"
Output.a = s - 1
```

```text
[JM5206] Ln 2, Col 12: Unsupported binary operation - for string and number
```

```jyro
Output.a = -"x"
```

```text
[JM5207] Ln 1, Col 12: Unsupported unary operation - for string
```

`+` never joins strings; strings are joined with `&`. A string operand of `+`, `+=`, `++`, or `--` is refused when the script is compiled, with JM3301 `StringOperandInAddition`, when the operand is a string literal or a variable declared with the type `string` or initialized with a string literal. A string that arrives at `+` in any other way is refused when the script runs, with JM5210 `StringOperandInAdditionRuntime`.

```jyro
Output.a = "a" + 1
```

```text
[JM3301] Ln 1, Col 12: The '+' operator does not accept a string operand
```

```jyro
# input: { "s": "a" }
Output.a = Input.s + 1
```

```text
[JM5210] Ln 2, Col 12: The '+' operator requires numeric operands but received a string
```
