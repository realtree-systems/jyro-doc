# Switch

```jyro
# fragment
switch expression do
    case value, value then
        statements
    case value then
        statements
    default then
        statements
end
```

The subject expression is evaluated once. The case values are then evaluated and compared with the subject by `==`, in source order, and the block of the first case with an equal value runs. When no case matches, the `default` block runs, if present. After a block runs the switch ends; there is no fall-through.

A case may list several values separated by commas. A case value may be any expression. `default` is optional and must be the last case. A switch with no cases is valid.

```jyro
# input: { "status": "created" }
switch Input.status do
    case "ok", "created" then
        Output.state = "success"
    case "error" then
        Output.state = "failure"
    default then
        Output.state = "unknown"
end
```

```json
{ "state": "success" }
```

Comparison uses the rules of `==`, so a number does not match a string of the same digits, and `null` matches only a `null` case.

```jyro
var results = []
foreach v in [1, "1", 2, null, 9] do
    switch v do
        case 1, 2 then
            results = Append(results, "one or two")
        case "1" then
            results = Append(results, "string one")
        case null then
            results = Append(results, "null")
        default then
            results = Append(results, "other")
    end
end
Output.results = results
```

```json
{ "results": ["one or two", "string one", "one or two", "null", "other"] }
```

Each case block is a scope. `break` inside a switch applies to the enclosing loop, not to the switch.

A `switch` compares values. A `match` statement takes apart a union variant; see [Match Statement](/docs/language/unions/match-statement).
