# Type Coercion

A value is converted to another type only where a type is declared: in a `var` declaration with a type, in an assignment with a type, in a typed function parameter, and in a typed union field. Operators never convert their operands, except that `&` takes the string form of each operand.

A value that is already of the declared type is unchanged. `any` accepts every value. Otherwise the conversion follows this table.

| Declared type | From | Result |
| --- | --- | --- |
| number | string | The number the text denotes, when the text has the form of a JSON number; otherwise an error |
| number | boolean | `1` for `true`, `0` for `false` |
| string | number | The number's string form |
| string | boolean | `"true"` or `"false"` |
| boolean | number | The number's truthiness |
| boolean | string | `true` for `"true"` and `false` for `"false"`, compared without regard to ASCII case; any other text is an error |

Every other conversion is a runtime error with JM5100 `InvalidType`, including `null` to any type other than null or any, and any conversion to array, object, null, or function.

```jyro
var n: number = "1e2"
var m: number = true
var s: string = 12
var b: boolean = "TRUE"
var c: boolean = 2
Output.values = [n, m, s, b, c]
```

```json
{ "values": [100, 1, "12", true, true] }
```

```jyro
var n: number = "abc"
```

```text
[JM5100] Ln 1, Col 1: Cannot give a value of type string to a target of type number
```

A declared type on a variable applies to every later assignment to that variable, including compound assignment and increment. See [Declaration](/docs/language/variables/declaration).
