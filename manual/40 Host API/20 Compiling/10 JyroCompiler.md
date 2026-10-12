# JyroCompiler

`JyroCompiler` compiles Jyro source to a `CompiledProgram`. It is a builder: each `With` and `Add` method changes the compiler and returns it. A compiler is not thread-safe and is used for one configuration.

```csharp
using RealTree.Jyro;

CompiledProgram program = new JyroCompiler()
    .WithStdlib()
    .WithSource("Output.total = Sum(Input.values)")
    .WithMaxEffect(FunctionEffect.Pure)
    .CompileOrThrow();
```

## Members

| Member | Effect |
| --- | --- |
| `WithSource(string)` | The script to compile. Required. |
| `WithStdlib()` | Makes the standard library available to the script. Without it, no standard library function can be called. |
| `AddFunction(IJyroFunction)`, `AddFunctions(IEnumerable<IJyroFunction>)` | Registers host functions. See [Registration](/docs/host-api/host-functions/registration). |
| `AddLibrarySource(string)`, `AddLibrary(JyroLibrary)` | Attaches a library. See [Libraries](/docs/host-api/compiling/libraries). |
| `WithMaxEffect(FunctionEffect)` | The highest effect the script may call. Default `Mutating`. See [Effects](/docs/host-api/host-functions/effects). |
| `WithLibraryMaxEffect(FunctionEffect)` | The highest effect a library body may call. Default `NonDeterministic`. |
| `WithMaxCompilationTime(TimeSpan)` | The time limit for compilation. Default 5 seconds; `TimeSpan.MaxValue` for none; otherwise more than zero and at most 24 hours. |
| `WithStats(JyroPipelineStats)` | Records the time of each stage. See [Results](/docs/host-api/running/results). |
| `Compile()` | Returns `JyroResult<CompiledProgram>`. |
| `CompileOrThrow()` | Returns the `CompiledProgram`, or throws `JyroException`. |
| `CompileToArtifact()` | Returns `JyroResult<byte[]>` holding a `.jyrx` artifact. See [Artifacts](/docs/host-api/compiling/artifacts). |

## Compilation

`Compile` runs the Parse, Validate, Link, and Compile stages. A failure in any stage ends the compilation, and the result holds every diagnostic that stage reported. With no source set, the result is JM6001 `NoSourceProvided`.

```csharp
using RealTree.Jyro;

var result = new JyroCompiler()
    .WithStdlib()
    .WithSource("Output.a = Length(")
    .Compile();

if (!result.IsSuccess)
{
    foreach (var message in result.Messages)
        Console.Error.WriteLine(DiagnosticFormatter.FormatMessage(message));
}
```

Compilation runs on a thread of the engine's own with a 16 MiB stack, so a source nested to the maximum depth compiles whatever the caller's stack. When the compilation time limit is reached, the work is abandoned and the result is JM6003 `CompilationTimeLimitExceeded`, with the stage that was running. A `JyroRuntimeException` thrown by a host function's `Name`, `Signature`, `Effect`, or `CheckCall` during compilation is reported in the result; any other exception from them is reported as JM6000 `UnknownCompilerError`.

On single-threaded browser WebAssembly, compilation runs on the caller's thread and the time limit is not enforced.

## CompiledProgram

`CompiledProgram` has no public members. It is immutable and may be shared across threads and executed concurrently. It holds the host functions it was linked with; a runner needs no function set. It records neither the resource limits nor the effect ceiling in force when it was compiled.
