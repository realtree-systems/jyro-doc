# Exit

```jyro
# fragment
exit
exit expression
```

`exit` ends the run successfully. The result of the run is the value of `Output` at that point. `exit` may be used at the top level, inside any block, and inside a function body.

```jyro
# input: { "items": [] }
Output.count = Length(Input.items)
if Output.count == 0 then
    exit
end
Output.first = Input.items[0]
```

```json
{ "count": 0 }
```

With an expression, the string form of the value is made available to the host as the run's exit message. A string is used as it is, a number or a boolean as its string form, and an array or an object as JSON. The exit message is not part of `Output` and is not a diagnostic.

```jyro
Output.done = true
exit "no items to process"
```

```text
Exit message: no items to process
```

```jyro
exit { code: 7 }
```

```text
Exit message: {"code":7}
```

A function value cannot be an exit message and is refused with JM5100 `InvalidType`.

The expression must start on the same line as `exit`; see [Source Text](/docs/language/lexical-structure/source-text). The host reads the message from the result of the run; see [Results](/docs/host-api/running/results).
