# Host API Overview

The `RealTree.Jyro` NuGet package contains the parser, the compiler, the standard library, and the host API. It targets .NET 8.0 and depends on FParsec only. Every public type is in the `RealTree.Jyro` namespace, and no F# type appears on the public surface.

```bash
dotnet add package RealTree.Jyro
```

## The pipeline

A script passes through these stages.

| Stage | Performed by | Work |
| --- | --- | --- |
| Parse | `JyroCompiler`, `JyroLibrary.Parse` | Source text to a syntax tree; lexical and syntax errors |
| Validate | `JyroCompiler`, `JyroArtifactLoader` | The static rules of the language |
| Link | `JyroCompiler`, `JyroArtifactLoader` | Each call resolved to a function; argument counts, names, and effects checked |
| Compile | `JyroCompiler`, `JyroArtifactLoader` | The tree compiled to a .NET delegate; an artifact written |
| Deserialize | `JyroArtifactLoader` | A `.jyrx` artifact read in place of Parse |
| Execute | `JyroRunner` | The delegate run against an input |

Three classes drive the stages. `JyroCompiler` produces a `CompiledProgram` from source. `JyroArtifactLoader` produces one from a `.jyrx` artifact. `JyroRunner` executes one.

```csharp
using RealTree.Jyro;

CompiledProgram program = new JyroCompiler()
    .WithStdlib()
    .WithSource("Output.greeting = 'Hello, ' & Input.name")
    .CompileOrThrow();

JyroExecutionResult result = new JyroRunner(program)
    .WithInputJson("""{ "name": "Ada" }""")
    .Execute();

if (result.TryGetValue(out JyroValue output))
    Console.WriteLine(output.ToJson()); // {"greeting":"Hello, Ada"}
```

## Compile once, run many times

A `CompiledProgram` holds the compiled script, the host functions it was linked with, and nothing else. It records no resource limit and no effect ceiling. A `JyroRunner` holds the input, the output seed, the limits, the cancellation token, and the other settings of one run. A program is compiled once and executed by as many runners as needed, each with its own input and limits.

| Belongs to the compilation | Belongs to the run |
| --- | --- |
| The source | The input |
| Libraries | The starting value of `Output` |
| Host functions | Resource limits |
| Effect ceilings | Cancellation |
| Compilation time limit | Host state, result observer, statistics |

## Results and exceptions

Each terminal method returns a `JyroResult<T>` or throws. `Compile`, `CompileToArtifact`, `Load`, `Execute`, and `JyroLibrary.Parse` return a result whose `IsSuccess`, `Messages`, and `TryGetValue` report the outcome. `CompileOrThrow`, `ExecuteOrThrow`, and `GetValueOrThrow` throw a `JyroException` holding the same messages. See [Results](/docs/host-api/running/results) and [Diagnostics](/docs/host-api/diagnostics).

A `null` argument to a method of `JyroCompiler`, `JyroArtifactLoader`, or `JyroRunner` throws `ArgumentNullException` and leaves the builder unchanged, except `WithInput(JyroValue)`, where `null` is the Jyro `null`, and `WithHostState`, where `null` is no state. An out-of-range limit throws `ArgumentOutOfRangeException`.

## Threads

A run executes synchronously on the thread that calls `Execute`. Compilation, loading, and `JyroLibrary.Parse` run on a thread of the engine's own with a 16 MiB stack, except on single-threaded browser WebAssembly, where they run on the caller's thread and the compilation time limit is not enforced.

| Type | Concurrent use |
| --- | --- |
| `CompiledProgram` | Immutable. Shared and executed concurrently. |
| `JyroCompiler`, `JyroArtifactLoader` | Not thread-safe. One per configuration. |
| `JyroRunner` | Not thread-safe. One per thread; it may be executed more than once. |
| `JyroExecutionContext` | Created by each `Execute` for that run. |
| `IJyroFunction` instances | Baked into the program and invoked concurrently from every run. They must hold no mutable state; per-run state goes through `WithHostState`. |
| `JyroValue` graphs | Mutable and not synchronized. Each run gets its own input, or the shared graph is frozen with `Freeze()`. |

## IL trimming

The compiler builds LINQ expression trees that reach engine members through reflection. A project published with IL trimming, which is on by default for Blazor WebAssembly and .NET MAUI, must root the Jyro assemblies, or scripts that use lambdas fail at run time.

```xml
<ItemGroup>
  <TrimmerRootAssembly Include="Jyro.Core" />
  <TrimmerRootAssembly Include="Jyro.Parser" />
  <TrimmerRootAssembly Include="Jyro.Compiler" />
  <TrimmerRootAssembly Include="Jyro.Stdlib" />
  <TrimmerRootAssembly Include="Jyro.Api" />
</ItemGroup>
```
