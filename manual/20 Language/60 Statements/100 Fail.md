# Fail

```jyro
# fragment
fail
fail expression
```

`fail` ends the run with a failure. The run reports the diagnostic JM5004 `ScriptFailure`, located at the `fail` statement. With an expression, the string form of the value is the message of the diagnostic; an array or an object is written as JSON. Without an expression, the message is `Script execution failed`.

```jyro
# input: { "qty": -1 }
Output.checked = true
if Input.qty < 0 then
    fail "Quantity must not be negative"
end
Output.qty = Input.qty
```

```text
[JM5004] Ln 4, Col 5: Quantity must not be negative
```

```jyro
fail
```

```text
[JM5004] Ln 1, Col 1: Script execution failed
```

```jyro
fail { code: 7, reason: "stock" }
```

```text
[JM5004] Ln 1, Col 1: {"code":7,"reason":"stock"}
```

The value of `Output` at the point of failure is returned to the host with the failure, so the properties written before the `fail` are available. `fail` may be used at the top level, inside any block, and inside a function body.

The expression must start on the same line as `fail`; see [Source Text](/docs/language/lexical-structure/source-text).
