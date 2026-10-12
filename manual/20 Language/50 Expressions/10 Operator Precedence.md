# Operator Precedence

Operators are listed from the tightest binding to the loosest. Operators on the same level are applied in the order given by the associativity column.

| Level | Operators | Associativity |
| --- | --- | --- |
| 1 | Call `f(args)`, property access `.name`, index `[i]`, postfix `++` and `--` | Left |
| 2 | Prefix `++` and `--`, unary `-` | Right |
| 3 | `*`, `/`, `%` | Left |
| 4 | `+`, `-` | Left |
| 5 | `&` | Left |
| 6 | `<`, `<=`, `>`, `>=` | Left |
| 7 | `==`, `!=` | Left |
| 8 | `is`, `is not` | Not chainable |
| 9 | `not` | Right |
| 10 | `and` | Left |
| 11 | `or` | Left |
| 12 | `??` | Right |
| 13 | `? :` | Right |
| 14 | Lambda `params => body` | The body extends as far as possible |

Parentheses group an expression and override the table. Assignment is a statement and does not appear in the table.

```jyro
var x = 5
Output.a = "a" & 1 + 2
Output.b = 1 & 2 < "2"
Output.c = not 1 == 2
Output.d = not 1 is string
Output.e = 0 ?? 1 or 2
Output.f = 1 ?? 2 == 2
Output.g = true ? 1 : false ? 2 : 3
Output.h = - 2 + 3
Output.i = 2 - -3
Output.j = -x++
Output.k = x
```

```json
{ "a": "a3", "b": true, "c": true, "d": true, "e": 0, "f": 1, "g": 1, "h": 1, "i": 5, "j": -5, "k": 6 }
```

| Expression | Read as |
| --- | --- |
| `"a" & 1 + 2` | `"a" & (1 + 2)` |
| `1 & 2 < "2"` | `(1 & 2) < "2"` |
| `not 1 == 2` | `not (1 == 2)` |
| `not 1 is string` | `not (1 is string)` |
| `0 ?? 1 or 2` | `0 ?? (1 or 2)` |
| `1 ?? 2 == 2` | `1 ?? (2 == 2)` |
| `true ? 1 : false ? 2 : 3` | `true ? 1 : (false ? 2 : 3)` |
| `-x++` | `-(x++)` |

A relational operator cannot be chained: `1 < 2 < 3` is read as `(1 < 2) < 3`, which compares a boolean with a number and is a runtime error. `1 is number is boolean` is a parse error.
