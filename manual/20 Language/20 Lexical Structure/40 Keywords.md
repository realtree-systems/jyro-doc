# Keywords

The following 38 words are reserved and cannot be used as identifiers.

| | | | | |
| --- | --- | --- | --- | --- |
| `and` | `array` | `boolean` | `break` | `by` |
| `case` | `continue` | `default` | `delete` | `do` |
| `downto` | `else` | `elseif` | `end` | `exit` |
| `fail` | `false` | `for` | `foreach` | `func` |
| `if` | `in` | `is` | `match` | `not` |
| `null` | `number` | `object` | `or` | `return` |
| `string` | `switch` | `then` | `to` | `true` |
| `union` | `var` | `while` | | |

Keywords are case-sensitive. `If`, `TRUE`, and `Null` are ordinary identifiers.

## Contextual keywords

`any` and `function` are type names, read only after `:` in a declaration or after `is` in a type check. Elsewhere they are ordinary identifiers.

```jyro
var any = 1
var function = 2
Output.sum = any + function
Output.check = Output.sum is any
```

```json
{ "sum": 3, "check": true }
```

## Reserved words as names

A reserved word is accepted as a property name after `.`, as a key in an object literal, and as the label of a named argument.

```jyro
Output.for = 1
Output.end = { default: true }
Output.keys = Keys({ if: 1, then: 2 })
```

```json
{ "for": 1, "end": { "default": true }, "keys": ["if", "then"] }
```
