# String Concatenation

The `&` operator joins two values into a string. Each operand is converted to its string form.

| Operand | String form |
| --- | --- |
| string | The string |
| number | The number's string form; see [Numbers](/docs/language/types-and-values/numbers) |
| boolean | `true` or `false` |
| null | `null` |
| array, object, function | A runtime error |

```jyro
Output.a = "Total: " & 42
Output.b = "Rate: " & 0.5 & "%"
Output.c = "Flag: " & true
Output.d = "Value: " & null
Output.e = 1 & 2
```

```json
{ "a": "Total: 42", "b": "Rate: 0.5%", "c": "Flag: true", "d": "Value: null", "e": "12" }
```

`&` binds more tightly than the comparison operators and less tightly than `+` and `-`, so `"n=" & 1 + 2` gives `"n=3"`.

An array, an object, or a function operand is refused with JM5209 `ConcatWithInvalidType`. The standard library function `ToJson` converts such a value to text.

```jyro
Output.a = [1] & "x"
```

```text
[JM5209] Ln 1, Col 12: & cannot convert array to string
```

The compound assignment `&=` appends to a variable, a property, or an element.

```jyro
var s = "a"
s &= "b"
s &= 1
Output.s = s
```

```json
{ "s": "ab1" }
```
