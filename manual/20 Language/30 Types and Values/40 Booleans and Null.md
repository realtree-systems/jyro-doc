# Booleans and Null

The boolean values are `true` and `false`. The value `null` is the only value of type null. A variable declared without an initializer, a missing property, an index outside an array, and a function that returns without a value all give `null`.

## Truthiness

The condition of `if`, `elseif`, `while`, and the conditional operator, and the operands of `and`, `or`, and `not`, are evaluated for truthiness. A value need not be a boolean.

| Value | Truthiness |
| --- | --- |
| `null` | false |
| `false` | false |
| `true` | true |
| A number equal to zero | false |
| Any other number | true |
| The empty string `""` | false |
| Any other string | true |
| Any array, including `[]` | true |
| Any object, including `{}` | true |
| Any function | true |

```jyro
Output.a = [not null, not 0, not "", not 0.0]
Output.b = [not [], not {}, not "0", not -1]
```

```json
{ "a": [true, true, true, true], "b": [false, false, false, false] }
```

The standard library function `ToBoolean` returns the truthiness of a value as a boolean.

## Null in comparisons

`null` is equal only to `null`. It is not equal to `false`, `0`, or `""`.

```jyro
Output.a = null == null
Output.b = null == false
Output.c = null == 0
Output.d = null == ""
```

```json
{ "a": true, "b": false, "c": false, "d": false }
```

The `??` operator substitutes a value for `null` only; see [Null Coalescing](/docs/language/expressions/null-coalescing).
