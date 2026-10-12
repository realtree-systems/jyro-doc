# Results

## JyroResult&lt;T&gt;

Every terminal method of the pipeline returns a `JyroResult<T>`: `Compile` and `Load` a `JyroResult<CompiledProgram>`, `CompileToArtifact` a `JyroResult<byte[]>`, `JyroLibrary.Parse` a `JyroResult<JyroLibrary>`, and `Execute` a `JyroExecutionResult`, which is a `JyroResult<JyroValue>`.

| Member | Content |
| --- | --- |
| `IsSuccess` | `true` when no message has severity Error |
| `Messages` | The diagnostics, an `IReadOnlyList<DiagnosticMessage>` |
| `HasValue` | Whether a value is present |
| `TryGetValue(out T value)` | The value, when present |
| `GetValueOrThrow()` | The value when `IsSuccess`; otherwise throws `JyroException` holding `Messages` |

A value is always present when `IsSuccess` is `true`. A value can also be present on failure: a run that writes to `Output` and then fails keeps the partial `Output`. A value is absent only when execution did not start, as for input that is not valid JSON. There is no `Value` property.

```csharp
using RealTree.Jyro;

CompiledProgram program = new JyroCompiler()
    .WithSource("Output.a = 1 Output.b = 1 / 0")
    .CompileOrThrow();

var result = new JyroRunner(program).Execute();

if (!result.IsSuccess)
{
    foreach (var message in result.Messages)
        Console.Error.WriteLine(DiagnosticFormatter.FormatMessage(message));
    // [JM5200] Ln 1, Col 25: Division by zero
}

if (result.TryGetValue(out JyroValue partial))
    Console.WriteLine(partial.ToJson()); // {"a":1}
```

A host can construct results. `JyroResult<T>.Success(value)` and `Success(value, messages)` make a success; the second throws `ArgumentException` when a message has severity Error. `Failure(message)`, `Failure(messages)`, `Failure(value, message)`, and `Failure(value, messages)` make a failure, with or without a partial value.

## JyroExecutionResult

The result of a run adds `ExitMessage`: the text of the value given to `exit`, or `null` when the script ended any other way. `exit ""` gives the empty string. The exit message is not a diagnostic and is not in `Messages`.

| Ending | `IsSuccess` | Value | `ExitMessage` |
| --- | --- | --- | --- |
| The last statement completes | `true` | `Output` | `null` |
| `exit` | `true` | `Output` | The value's text, or `null` without a value |
| `fail`, a runtime error, a limit, cancellation | `false` | `Output` as it stood | `null` |
| Input could not be converted | `false` | None | `null` |

Every diagnostic of a run has stage Execute.

## Reading the value

| Member | Returns |
| --- | --- |
| `ToJson()` | The value as JSON text |
| `WriteJson(Utf8JsonWriter)` | The value written to a `System.Text.Json` writer |
| `ToObjectValue()` | A .NET object: `null`, `bool`, `long`, `decimal`, `double`, `string`, `List<object>`, or `Dictionary<string, object>` |
| `ToStringValue()`, `ToDouble()`, `ToInt32()`, `ToInt64()` | The scalar, or `InvalidCast` for a value of another type |
| `AsObject()`, `AsArray()` | The container, for its members |

See [JyroValue](/docs/host-api/values/jyrovalue).

## JyroPipelineStats

A `JyroPipelineStats` passed with `WithStats` to the compiler, the loader, or the runner records what that path ran. The same instance may be given to both halves.

| Path | Records |
| --- | --- |
| `Compile`, `CompileToArtifact` | `ParseTime`, `ValidateTime`, `LinkTime`, `CompileTime` |
| `Load` | `DeserializeTime`, `ValidateTime`, `LinkTime`, `CompileTime` |
| `Execute` | `ExecuteTime`, `StatementCount`, `LoopIterations`, `PeakCallDepth`, `LimiterElapsedTime` |

`TotalTime` is the sum of the six stage times. A stage a path did not run reads as zero. The properties are read-only; only the pipeline writes them.

```csharp
using RealTree.Jyro;

var stats = new JyroPipelineStats();

CompiledProgram program = new JyroCompiler()
    .WithStdlib()
    .WithSource("Output.n = Length(Input)")
    .WithStats(stats)
    .CompileOrThrow();

var result = new JyroRunner(program)
    .WithInputJson("[1, 2, 3]")
    .WithStats(stats)
    .Execute();

Console.WriteLine($"{stats.ParseTime.TotalMilliseconds} ms parse, {stats.ExecuteTime.TotalMilliseconds} ms execute, {stats.StatementCount} statements");
```
