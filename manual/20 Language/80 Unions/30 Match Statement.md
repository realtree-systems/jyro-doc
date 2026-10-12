# Match Statement

```jyro
# fragment
match expression do
    case Variant(binding, binding) then
        statements
    case Variant then
        statements
end
```

A `match` statement runs the block of the case that names the variant of the subject. The bindings of a case are positional: each binding names a variable that receives the corresponding field of the variant, in declaration order. A case for a variant without fields is written without parentheses, or with empty parentheses.

```jyro
union Shape
    Circle(radius: number)
    Rect(width: number, height: number)
    Empty
end
var areas = []
foreach s in [Circle(2), Rect(3, 4), Empty()] do
    match s do
        case Circle(radius) then
            areas = Append(areas, 3 * radius * radius)
        case Rect(w, h) then
            areas = Append(areas, w * h)
        case Empty then
            areas = Append(areas, 0)
    end
end
Output.areas = areas
```

```json
{ "areas": [12, 12, 0] }
```

Each case block is a scope, and the bindings are variables of that scope.

## Rules

The cases of a match belong to one union: the union of the first case that names a declared variant. The match must have a case for every variant of that union, each variant at most once, with as many bindings as the variant has fields. A `default` case is not allowed.

| Rule | Diagnostic |
| --- | --- |
| A variant of the union has no case | JM3602 `MatchNotExhaustive` |
| A variant has two cases | JM3603 `MatchDuplicateCase` |
| A case names a variant that is not in the union | JM3606 `MatchUnknownVariant` |
| The number of bindings differs from the number of fields | JM3604 `MatchBindingCountMismatch` |
| A name is bound twice in one case | JM3611 `MatchDuplicateBinding` |

```jyro
union U A B end
match A() do
    case A then
        Output.a = 1
end
```

```text
[JM3602] Ln 2, Col 1: Match on union 'U' is not exhaustive; missing case(s): B
```

```jyro
union U A(x) B end
match A(1) do
    case A(x, y) then
        Output.a = 1
    case B then
        Output.a = 2
end
```

```text
[JM3604] Ln 3, Col 5: Case 'A' has 2 binding(s) but variant 'A' has 1 field(s)
```

A `default` case in a match statement is a parse error.

## Runtime

The subject is evaluated once. When it is an object, its `_variant` property selects the case. A subject that is not an object, or whose `_variant` names no case of the match, is a runtime error.

```jyro
union U A B end
match { x: 1 } do
    case A then
        Output.a = 1
    case B then
        Output.a = 2
end
```

```text
[JM5403] Ln 2, Col 1: The subject of a match is not a value of a variant that one of its cases names
```
