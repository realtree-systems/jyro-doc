# Declaring Unions

```jyro
# fragment
union Name
    Variant(field, field: type)
    Variant
end
```

A union is a named set of variants. Each variant has a name and zero or more fields, and each field may have a type. A variant with no fields is written without parentheses, or with empty parentheses. A union must have at least one variant. The declaration may be written on one line.

```jyro
union Shape
    Circle(radius: number)
    Rect(width: number, height: number)
    Empty
end
union Direction Up Down end
Output.circle = Circle(2)
Output.up = Up()
```

```json
{ "circle": { "_variant": "Circle", "radius": 2 }, "up": { "_variant": "Up" } }
```

## Placement and visibility

A union must be declared at the top level of the script. It is visible throughout the script, including before its declaration, and to the bodies of attached libraries.

```jyro
if true then
    union U A end
end
```

```text
[JM3600] Ln 2, Col 5: Union 'U' must be declared at the top level
```

```jyro
func F()
    union U A end
    return 1
end
```

```text
[JM3605] Ln 2, Col 5: Union declarations cannot be inside functions
```

## Names

| Conflict | Diagnostic |
| --- | --- |
| Two unions with the same name | JM3608 `DuplicateUnionName` |
| Two variants with the same name, in the same or in different unions | JM3601 `DuplicateVariantName` |
| Two fields with the same name in one variant | JM3610 `DuplicateFieldName` |
| A field named `_variant` | JM3609 `ReservedFieldName` |
| A variant named `Input` or `Output` | JM3103 `ReservedName` |
| A variant with the name of a standard library or host function | JM4300 `VariantShadowsFunction` |
| A variant with the name of a function declared with `func` | JM4301 `FunctionShadowsVariant` |

Variant names are shared across every union in the compilation, since a variant is constructed by its name alone.

```jyro
union Shape
    Circle(radius: number)
end
union Sizes
    Circle(d: number)
end
```

```text
[JM3601] Ln 5, Col 5: Variant 'Circle' is already defined in union 'Shape'
```

```jyro
union U A(_variant) end
```

```text
[JM3609] Ln 1, Col 1: Field name '_variant' in variant 'A' is reserved for the variant tag
```

```jyro
union U Length end
```

```text
[JM4300] Ln 1, Col 1: Variant constructor 'Length' shadows built-in function 'Length'
```
