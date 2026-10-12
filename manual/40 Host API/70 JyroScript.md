# JyroScript

`JyroScript` holds two static methods that compile a script with the standard library and run it once. Each call compiles again; a script that runs more than once is compiled with `JyroCompiler` and executed with `JyroRunner`.

| Method | Input |
| --- | --- |
| `JyroScript.ExecuteScript(string source, object data)` | A .NET value, converted as `JyroRunner.WithInput(object)` converts it |
| `JyroScript.ExecuteScriptWithJson(string source, string json)` | JSON text |

Both return a `JyroExecutionResult`. A compilation error is reported in the result's `Messages` with its own stage; a run error has stage Execute. Neither method throws for a fault in the script or the input.

```csharp
using RealTree.Jyro;

JyroExecutionResult result = JyroScript.ExecuteScriptWithJson(
    "Output.total = Sum(Input.prices)",
    """{ "prices": [1.5, 2.25] }""");

if (result.TryGetValue(out JyroValue output))
    Console.WriteLine(output.ToJson()); // {"total":3.75}
```

```csharp
using RealTree.Jyro;

var result = JyroScript.ExecuteScript("Output.doubled = Input.n * 2", new { n = 21 });
Console.WriteLine(result.GetValueOrThrow().ToJson()); // {"doubled":42}
```

The run uses `JyroExecutionOptions.Default`. From F#, the same two functions are `executeScript` and `executeScriptWithJson` in the `JyroScript` module.
