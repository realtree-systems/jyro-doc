# Resource Limits

Every run is bounded. The limits are on by default: a runner configured with nothing runs at `JyroExecutionOptions.Default`. The limits belong to the run and are not recorded in a `CompiledProgram` or an artifact; one program serves every policy.

| Limit | Default | Setter | When exceeded |
| --- | --- | --- | --- |
| Execution time | 5 seconds | `WithMaxExecutionTime(TimeSpan)` | JM5903 `ExecutionTimeLimitExceeded` |
| Statements executed | 10,000 | `WithMaxStatements(int)` | JM5900 `StatementLimitExceeded` |
| Loop iterations, cumulative across the run | 1,000 | `WithMaxLoopIterations(int)` | JM5901 `LoopIterationLimitExceeded` |
| Call depth | 64 | `WithMaxCallDepth(int)` | JM5902 `CallDepthLimitExceeded` |
| One regular expression evaluation | 1 second | `WithMaxRegexTime(TimeSpan)` | JM5700 `RegexTimeout` |
| Host cancellation | none | `WithCancellationToken(CancellationToken)` | JM5003 `CanceledByHost` |
| Compilation time | 5 seconds | `JyroCompiler.WithMaxCompilationTime`, `JyroArtifactLoader.WithMaxCompilationTime`, `JyroLibrary.Parse(source, time)` | JM6003 `CompilationTimeLimitExceeded` |

A run that reaches a limit ends with a failed result whose value is `Output` as it stood.

```csharp
using RealTree.Jyro;

CompiledProgram program = new JyroCompiler()
    .WithSource("var n = 0 while true do n++ end")
    .CompileOrThrow();

var result = new JyroRunner(program)
    .WithMaxExecutionTime(TimeSpan.FromSeconds(1))
    .WithMaxLoopIterations(500)
    .Execute();

// result.Messages[0].Code is MessageCode.LoopIterationLimitExceeded
```

## JyroExecutionOptions

`JyroExecutionOptions` holds the five run limits as settable properties, so it can be built with an object initializer or bound from configuration. `WithExecutionOptions` applies all of them.

```csharp
using RealTree.Jyro;

var options = new JyroExecutionOptions
{
    MaxExecutionTime = TimeSpan.FromSeconds(10),
    MaxStatements = 20_000,
    MaxLoopIterations = 2_000,
    MaxCallDepth = 96,
    MaxRegexTime = TimeSpan.FromSeconds(2),
};

CompiledProgram program = new JyroCompiler().WithSource("Output.ok = true").CompileOrThrow();
var result = new JyroRunner(program).WithExecutionOptions(options).Execute();
```

| Rule | Detail |
| --- | --- |
| A count of zero | Not set; the default applies |
| A negative count | `ArgumentOutOfRangeException`; the runner keeps its limits |
| `MaxExecutionTime` | `TimeSpan.MaxValue` for no limit; otherwise more than zero and at most 24 hours. Zero is refused, so it is always set. |
| `MaxRegexTime` | More than zero and at most 24 hours; zero in the options means not set |
| `MaxCallDepth` above `MaxSafeCallDepth` (96) | Clamped to 96 without an error |

`JyroExecutionOptions.Default` holds the defaults in the table. `JyroExecutionOptions.Unlimited` removes the time, statement, and loop limits and keeps the call depth at 96. The two-argument form `new JyroExecutionOptions(TimeSpan, int, int, int)` takes the execution time, statements, loop iterations, and call depth. `Equals` compares those four limits.

## Call depth

`MaxSafeCallDepth` assumes a thread stack of 1 MB, the size of a Windows thread pool or ASP.NET request thread. At that depth a script cannot overflow the stack, including when the deepest call raises an error, runs `fail`, or runs `exit`. A host that executes scripts on a thread with a smaller stack sets `MaxCallDepth` lower. Every active call, including standard library and host function calls and the calls a standard library function makes to a lambda, occupies one level.

## What is counted

The limiter checks the time limit and the cancellation token at every statement and loop iteration. Call boundaries check the call depth. A standard library function that iterates a collection calls `CheckExecutionLimits` as it works, so a long call is bounded by the time limit, and counts as one statement.

## Cancellation

`WithCancellationToken` supplies a token that stops the run at the next statement, loop iteration, or standard library checkpoint. The limiter links it with its own token, which it cancels at the execution time limit, and host functions receive the combined token as `ctx.CancellationToken`. Blocking work inside a host function stops when the token fires, without waiting for the next checkpoint.

```csharp
using RealTree.Jyro;

CompiledProgram program = new JyroCompiler().WithSource("Output.ok = true").CompileOrThrow();

using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(10));
var result = new JyroRunner(program)
    .WithMaxExecutionTime(TimeSpan.FromSeconds(5))
    .WithCancellationToken(cts.Token)
    .Execute();
```

The time limit fires on time whatever the state of the thread pool: the engine keeps one deadline thread per process and cancels each run's token from threads of its own. The token governs execution only; compilation is bounded by the compilation time limit.

A run stopped by the host's token fails with JM5003 `CanceledByHost`; a run that spent its time fails with JM5903 `ExecutionTimeLimitExceeded`. The time limit is tested first, so a timeout is never reported as a cancellation. A host function that catches an `OperationCanceledException` tells the two apart through `ctx.HostCancellationRequested`.

## Fixed limits

The limits on value size (16,777,216), value nesting depth (128), and source nesting depth (256) are fixed and cannot be changed by the host. See [Limits](/docs/language/types-and-values/limits).
