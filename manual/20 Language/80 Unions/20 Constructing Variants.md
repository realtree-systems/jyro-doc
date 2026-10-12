# Constructing Variants

A variant value is made by calling the variant's name with one argument per field. The result is an object with a `_variant` property holding the variant's name, followed by one property per field in declaration order.

```jyro
union Shape
    Circle(radius: number)
    Rect(width: number, height: number)
    Empty
end
Output.c = Circle("2")
Output.r = Rect(3, 4)
Output.e = Empty()
Output.tag = Output.c._variant
Output.isObject = Output.c is object
```

```json
{
  "c": { "_variant": "Circle", "radius": 2 },
  "r": { "_variant": "Rect", "width": 3, "height": 4 },
  "e": { "_variant": "Empty" },
  "tag": "Circle",
  "isObject": true
}
```

A typed field converts its argument as described in [Type Coercion](/docs/language/types-and-values/type-coercion). Every field is required; the argument count is checked when the script is compiled. Arguments may be positional or named.

```jyro
union U A(x: number) end
Output.a = A()
```

```text
[JM4200] Ln 2, Col 12: Function 'A' requires at least 1 arguments, but 0 were provided
```

A variant with no fields must still be called with parentheses. The bare name is an undeclared variable.

```jyro
union U Empty end
Output.e = Empty
```

```text
[JM3100] Ln 2, Col 12: Undeclared variable 'Empty'
```

Each call makes a new object. The fields are ordinary properties and can be read and written. The value is taken apart with `match`; see [Match Statement](/docs/language/unions/match-statement). There is no `is` check for a variant; the `_variant` property identifies it.
