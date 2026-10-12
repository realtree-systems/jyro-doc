# Comparison Operators

| Operator | Operation | Operands |
| --- | --- | --- |
| `==` | Equal | Any two values |
| `!=` | Not equal | Any two values |
| `<` | Less than | Two numbers or two strings |
| `<=` | Less than or equal | Two numbers or two strings |
| `>` | Greater than | Two numbers or two strings |
| `>=` | Greater than or equal | Two numbers or two strings |

Every comparison gives a boolean. Equality compares arrays and objects by their contents and never raises an error; values of different types are never equal. The rules are given in [Equality and Ordering](/docs/language/types-and-values/equality-and-ordering).

```jyro
Output.a = 1 == 1.0
Output.b = "1" != 1
Output.c = [1, 2] == [1, 2]
Output.d = 2 >= 2
Output.e = "b" > "a"
```

```json
{ "a": true, "b": true, "c": true, "d": true, "e": true }
```

A relational operator with operands of different types, or of a type other than number or string, is refused with JM5206 `UnsupportedBinaryOperation`.

```jyro
Output.a = 1 < "2"
```

```text
[JM5206] Ln 1, Col 12: Unsupported binary operation < for number and string
```

The relational operators and the equality operators are on separate levels; see [Operator Precedence](/docs/language/expressions/operator-precedence). `1 < 2 == true` is read as `(1 < 2) == true`.
