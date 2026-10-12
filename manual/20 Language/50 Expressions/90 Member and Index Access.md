# Member and Index Access

A property of an object is read with `.` followed by a name, or with an expression in square brackets. An element of an array or a code unit of a string is read with an expression in square brackets. Accesses chain from left to right.

```jyro
# input: { "order": { "lines": [ { "sku": "A1", "qty": 2 } ] }, "key": "sku" }
Output.a = Input.order.lines[0].sku
Output.b = Input.order["lines"][0][Input.key]
Output.c = "text"[0]
```

```json
{ "a": "A1", "b": "A1", "c": "t" }
```

Any name may follow `.`, including a reserved keyword. A key that is not a valid identifier is written in brackets as a string.

## Reading

| Expression | Result |
| --- | --- |
| `o.name`, `o["name"]`, where `o` is an object with the property | The value |
| `o.name`, `o["name"]`, where `o` is an object without the property | `null` |
| `x.name`, where `x` is not an object | `null` |
| `a[i]`, where `a` is an array or a string and `i` is an integer in range | The element or code unit |
| `a[i]`, where `i` is an integer out of range or negative | `null` |
| `a[i]`, where `i` is a number with a fraction | JM5310 `IntegerIndexRequired` |
| `a[i]`, where `i` is not a number | `null` |
| `o[k]`, where `o` is an object and `k` is not a string | `null` |

A read never fails because a value is missing or is `null`. A path through missing properties evaluates to `null`.

```jyro
# input: { "a": { } }
Output.a = Input.a.b.c
Output.b = Input.a[0]
Output.c = [1, 2][5]
```

```json
{ "a": null, "b": null, "c": null }
```

## Writing

Property and index accesses are also the targets of assignment, compound assignment, increment, and `delete`. The rules are given in [Objects](/docs/language/types-and-values/objects) and [Arrays](/docs/language/types-and-values/arrays).

## Line ends

A `.` at the start of a line continues the expression on the line before it. A `[` at the start of a line starts a new statement, so an index must be written on the same line as the expression it applies to.

```jyro
var items = [10, 20]
Output.a = items
[0]
```

```json
{ "a": [10, 20] }
```
