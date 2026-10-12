# Runtime Failure

A run ends in one of three ways.

| Ending | Caused by | Result |
| --- | --- | --- |
| Success | The last statement completes, or `exit` runs | `Output`, and the exit message if `exit` was given a value |
| Failure | A runtime error, `fail`, a resource limit, a host function error, or cancellation by the host | One diagnostic with stage Execute, and `Output` as it was at that point |
| Not started | The input could not be converted, for example invalid JSON | One diagnostic, and no `Output` |

The first runtime error stops the run. No later statement runs, and no further diagnostic is reported.

## Partial output

The value of `Output` at the moment of failure is returned with the failure. Properties written before the error are present; the statement that failed has no effect.

```jyro
# input: { "items": [1, 2, 0, 4] }
var done = []
foreach n in Input.items do
    done = Append(done, 10 / n)
    Output.done = done
end
```

```text
[JM5200] Ln 4, Col 25: Division by zero
```

In this run, `Output` holds `{ "done": [10, 5] }` when the failure is reported.

## Sources of failure

| Source | Diagnostic |
| --- | --- |
| An operation on a value of the wrong type, an index or property write on the wrong kind of value, division by zero, and the other runtime rules described with each feature | The rule's code, JM5xxx |
| `fail` | JM5004 `ScriptFailure`, with the message given to `fail` |
| A resource limit | JM5900 to JM5903; see [Resource Limits](/docs/language/errors-and-limits/resource-limits) |
| A host function that reports an error | JM5006 `HostFunctionError` |
| A host function that throws any other exception | JM5002 `RuntimeError` |
| Cancellation by the host | JM5003 `CanceledByHost` |
| A value written to `Output` that cannot be converted to JSON, such as a string holding half of a surrogate pair | Reported when the host converts the result |

## Location

A runtime diagnostic is located at the start of the expression or statement that failed, in the source as compiled. A failure inside a function is located at the statement in the function body, not at the call.

```jyro
func Ratio(n)
    return 10 / n
end
Output.a = Ratio(0)
```

```text
[JM5200] Ln 2, Col 12: Division by zero
```
