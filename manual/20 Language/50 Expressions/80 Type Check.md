# Type Check

The expression `value is type` gives `true` when `value` is of the named type, and `value is not type` gives the opposite. The type is one of the type keywords: `null`, `boolean`, `number`, `string`, `array`, `object`, `function`, or `any`.

```jyro
# input: { "n": 1, "s": "a", "list": [1] }
Output.a = Input.n is number
Output.b = Input.s is not string
Output.c = Input.list is array
Output.d = Input.missing is null
Output.e = Input.n is any
Output.f = (x => x) is function
```

```json
{ "a": true, "b": false, "c": true, "d": true, "e": true, "f": true }
```

`is any` is always `true`. A value of a union variant is an object, so `variant is object` is `true`; the variant itself is tested with `match` or by reading its `_variant` property.

A type check cannot be chained: `x is number is boolean` is a parse error. `is` binds more tightly than `not`, so `not x is string` is read as `not (x is string)`. See [Operator Precedence](/docs/language/expressions/operator-precedence).
