# Declaration

A variable is declared with `var`, an identifier, an optional type, and an optional initializer.

```jyro
# fragment
var name
var name = expression
var name: type
var name: type = expression
```

A variable declared without an initializer has the value `null`. A variable must be declared before it is used.

```jyro
var a
var b = 2
var c: any
var d: number = 4
Output.values = [a, b, c, d]
```

```json
{ "values": [null, 2, null, 4] }
```

## Typed variables

A declared type is applied to the initializer and to every later assignment to the variable, including compound assignment and increment. A value of another type is converted as described in [Type Coercion](/docs/language/types-and-values/type-coercion), and a value that cannot be converted is a runtime error.

```jyro
var count: number = 42
count = "7"
count += 1
Output.count = count
Output.type = TypeOf(count)
```

```json
{ "count": 8, "type": "number" }
```

```jyro
var count: number = 42
count = "many"
```

```text
[JM5100] Ln 2, Col 1: Cannot give a value of type string to a target of type number
```

A variable declared with a type other than `any` must have an initializer.

```jyro
var n: number
```

```text
[JM3302] Ln 1, Col 1: Typed variable 'n' must have an initializer
```

## Names

A variable cannot be named `Input` or `Output`, and cannot have the name of a reserved keyword. A variable cannot be declared twice in the same scope; see [Scope](/docs/language/variables/scope).

```jyro
var Input = 1
```

```text
[JM3103] Ln 1, Col 1: Cannot bind 'Input'; Input and Output are supplied by the host
```
