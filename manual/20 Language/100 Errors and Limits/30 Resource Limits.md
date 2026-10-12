# Resource Limits

Every run is bounded by limits that the host sets. A run that reaches a limit ends with a failure and keeps the `Output` written so far.

| Limit | Default | Diagnostic |
| --- | --- | --- |
| Execution time | 5 seconds | JM5903 `ExecutionTimeLimitExceeded` |
| Statements executed | 10,000 | JM5900 `StatementLimitExceeded` |
| Loop iterations, counted across every loop in the run | 1,000 | JM5901 `LoopIterationLimitExceeded` |
| Call depth | 64; a host can set at most 96 | JM5902 `CallDepthLimitExceeded` |
| Time for one regular expression evaluation | 1 second | JM5700 `RegexTimeout` |
| Compilation time | 5 seconds | JM6003 `CompilationTimeLimitExceeded` |

A host may also cancel a run, which ends it with JM5003 `CanceledByHost`. The limits on value size and nesting are fixed and are listed in [Limits](/docs/language/types-and-values/limits). How a host sets the limits is described in [Resource Limits](/docs/host-api/running/resource-limits) in the Host API section.

## What is counted

Every executed statement counts toward the statement limit, including `break`, `continue`, and the statements of a function body. A call to a standard library function counts as one statement, however much work it does; a long-running standard library call is bounded by the time limit.

Every iteration of a `while`, `for`, or `foreach` loop counts toward the loop limit. The count is shared by all loops in the run, nested or not.

Every active call, including calls to standard library and host functions and the calls a standard library function makes to a lambda, occupies one level of call depth.

```jyro
var n = 0
for i in 0 to 2000 do
    n++
end
```

```text
[JM5901] Ln 2, Col 1: Script execution exceeded maximum loop iteration limit of 1000
```

```jyro
func Down(n)
    return Down(n + 1)
end
Output.a = Down(0)
```

```text
[JM5902] Ln 2, Col 12: Script execution exceeded maximum call depth limit of 64
```

The time limit is checked at statement boundaries and inside long-running standard library functions. A limit on time is reported before a cancellation by the host when both apply.
