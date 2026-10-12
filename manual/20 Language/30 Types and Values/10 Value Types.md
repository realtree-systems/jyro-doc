# Value Types

Every value is of one of seven types.

| Type | Values | Literal form |
| --- | --- | --- |
| null | The single value `null` | `null` |
| boolean | `true` and `false` | `true`, `false` |
| number | An exact decimal or an inexact double; always finite | `42`, `3.5`, `0xFF` |
| string | A sequence of UTF-16 code units | `"text"`, `'text'` |
| array | An ordered list of values of any type | `[1, "a"]` |
| object | An ordered map from string keys to values of any type | `{ a: 1 }` |
| function | A lambda value | `x => x + 1` |

A value of a union variant is an object; see [Unions](/docs/language/unions/declaring-unions).

## Type names

A type is named by a type keyword in a variable or parameter declaration, in an assignment, and in an `is` check. The keyword `any` names every type.

| Keyword | Type |
| --- | --- |
| `null` | null |
| `boolean` | boolean |
| `number` | number |
| `string` | string |
| `array` | array |
| `object` | object |
| `function` | function |
| `any` | any type |

The standard library function `TypeOf` returns the type name of a value as a string. Diagnostics name types the same way.

```jyro
Output.names = [TypeOf(null), TypeOf(true), TypeOf(1), TypeOf("s"), TypeOf([]), TypeOf({}), TypeOf(x => x)]
Output.check = [1 is number, "a" is string, [] is any, null is object]
```

```json
{
  "names": ["null", "boolean", "number", "string", "array", "object", "function"],
  "check": [true, true, true, false]
}
```
