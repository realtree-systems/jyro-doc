# JyroRunner

`JyroRunner` executes a `CompiledProgram` against an input under the limits of one run. It is a builder: each `With` method changes the runner and returns it. A runner is not thread-safe; it is cheap, and one is made per thread. A runner may be executed more than once, and each `Execute` starts from the runner's settings, not from what an earlier run left.

```csharp
using RealTree.Jyro;

CompiledProgram program = new JyroCompiler()
    .WithStdlib()
    .WithSource("Output.names = Map(Input.people, p => p.name)")
    .CompileOrThrow();

JyroExecutionResult result = new JyroRunner(program)
    .WithInputJson("""{ "people": [ { "name": "Ada" }, { "name": "Alan" } ] }""")
    .WithMaxExecutionTime(TimeSpan.FromSeconds(2))
    .Execute();
```

## Members

| Member | Effect |
| --- | --- |
| `JyroRunner(CompiledProgram)` | The program to run. |
| `WithInput(JyroValue)` | The value of `Input`. `null` is the Jyro `null`. |
| `WithInput(object)` | A .NET value converted to a Jyro value; see [JyroValue](/docs/host-api/values/jyrovalue). Carries `RequiresUnreferencedCode`: a trimmed host gets warning IL2026 at the call. |
| `WithInputJson(string)` | JSON text, parsed as strict RFC 8259 JSON. |
| `WithInputJson(Stream)` | UTF-8 JSON read once from the stream, at the next `Execute`. |
| `WithOutput(JyroValue)` | The starting value of `Output`. Each run starts from a copy. |
| `WithExecutionOptions(JyroExecutionOptions)` | All the limits at once. |
| `WithMaxExecutionTime`, `WithMaxStatements`, `WithMaxLoopIterations`, `WithMaxCallDepth`, `WithMaxRegexTime` | One limit each; see [Resource Limits](/docs/host-api/running/resource-limits). |
| `WithCancellationToken(CancellationToken)` | A token that ends the run when canceled. |
| `WithHostState(object)` | A value every host function of the run reads as `ctx.HostState`. `null` is no state. |
| `WithFunctionResultObserver(Action<string, JyroValue>)` | A callback invoked after each host function call with the function's name and result. |
| `WithStats(JyroPipelineStats)` | Records the run's time and counters. |
| `Execute()` | Returns `JyroExecutionResult`. |
| `ExecuteOrThrow()` | Returns the value of `Output`, or throws `JyroException`. |

## Input

Input set by any of the four methods replaces the input set before. With no input set, `Input` is `null`. The value is frozen before the run: the script reads it and cannot change it.

A fault in the input is reported by `Execute` as a failed result with no value, not thrown by the `With` method: text that is not JSON gives JM5917 `InvalidJson`, a NaN or an infinity in a .NET value gives JM5921 or JM5922, a .NET value with no Jyro form gives JM5923 `HostValueNotConvertible`, and a value nested deeper than 128 gives JM5910 `ValueNestingLimitExceeded`.

```csharp
using RealTree.Jyro;

CompiledProgram program = new JyroCompiler().WithSource("Output.n = Input.n").CompileOrThrow();

var result = new JyroRunner(program)
    .WithInputJson("{ not json }")
    .Execute();

// result.IsSuccess is false, result.HasValue is false, and
// result.Messages holds JM5917 InvalidJson.
```

## Output

`Output` starts as an empty object. `WithOutput` supplies another starting value; each run starts from a copy of it, taken when `Execute` is called, so no run changes the value given, and a container reached from `Input` stays read-only inside the copy. The script may replace `Output` with a value of any type. The final value of `Output` is the value of the result.

## Host state and observation

`WithHostState` passes a per-run object to every host function through `JyroExecutionContext.HostState`. The engine never reads it. A multi-tenant host gives each runner of one shared program its own tenant this way, rather than storing state on a function instance, which is shared by every run.

`WithFunctionResultObserver` attaches a callback to the run's context. It is invoked after every host function call returns, with the function's name and its result, and observes any program however it was compiled.

## Execution

`Execute` builds a fresh execution context and limiter, runs the program synchronously on the calling thread, and returns when the script ends, fails, reaches a limit, or is canceled. The result is described in [Results](/docs/host-api/running/results).

`ExecuteOrThrow` returns the value of `Output` on success and throws a `JyroException` holding the diagnostics otherwise.
