# Match Expression

```jyro
# fragment
match expression do
    case Variant(binding, binding) then expression
    case Variant then expression
end
```

A `match` expression evaluates to the body of the case that names the variant of the subject. Each case body is a single expression. The bindings are visible in that expression. The rules on cases are the same as for the [Match Statement](/docs/language/unions/match-statement).

```jyro
union Shape
    Circle(radius: number)
    Rect(width: number, height: number)
    Empty
end
func Describe(s)
    return match s do
        case Circle(radius) then "circle " & radius
        case Rect(w, h) then "rect " & w & "x" & h
        case Empty then "empty"
    end
end
Output.described = Map([Circle(2), Rect(3, 4), Empty()], s => Describe(s))
```

```json
{ "described": ["circle 2", "rect 3x4", "empty"] }
```

A match expression may appear wherever an expression is allowed, including after `return` and inside a lambda body. A statement that starts with `match` is always a match statement.

A `case default` in a match expression is refused with JM3607 `MatchWithDefault`.

```jyro
union U A B end
Output.a = match A() do
    case A then 1
    case B then 2
    case default then 3
end
```

```text
[JM3607] Ln 5, Col 5: 'default' is not allowed in a match
```
