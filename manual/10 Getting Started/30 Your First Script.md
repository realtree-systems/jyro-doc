# Your First Script

A script reads the data the host supplies through `Input`, and writes its result to `Output`. The script below reads a name and a list of scores, computes the average with the standard library function `Average`, and writes three properties.

Save the input as `input.json`:

```json
{ "name": "Ada", "scores": [72, 88, 95] }
```

Save the script as `hello.jyro`:

```jyro
# input: { "name": "Ada", "scores": [72, 88, 95] }
var average = Average(Input.scores)

Output.greeting = "Hello, " & Input.name
Output.average = average
Output.grade = average >= 80 ? "pass" : "fail"
```

```json
{
  "greeting": "Hello, Ada",
  "average": 85,
  "grade": "pass"
}
```

## Running the script from the command line

```bash
jyro run -s hello.jyro -i input.json
```

The tool compiles the script, runs it with the contents of `input.json` as `Input`, and prints `Output` as JSON indented by two spaces.

## Running the script from a .NET application

The same script is compiled with `JyroCompiler` and run with `JyroRunner`. The compiled program is reused for every run; the runner holds the input for one run.

```csharp
using RealTree.Jyro;

var source = File.ReadAllText("hello.jyro");

CompiledProgram program = new JyroCompiler()
    .WithStdlib()
    .WithSource(source)
    .CompileOrThrow();

var result = new JyroRunner(program)
    .WithInputJson(File.ReadAllText("input.json"))
    .Execute();

if (result.TryGetValue(out var output))
    Console.WriteLine(output.ToJson());
```

`CompileOrThrow` throws a `JyroException` when the script does not compile. `Execute` returns a result whose `IsSuccess` is false when the run fails, and whose `Messages` list holds the diagnostics.

## Reading a diagnostic

A runtime error ends the run and is reported as a diagnostic with a code, a position, and a message. The script below divides by zero on its second line.

```jyro
var count = 0
Output.ratio = 10 / count
```

```text
[JM5200] Ln 2, Col 16: Division by zero
```

The code identifies the error, and the position is the line and column at which the failing expression starts. The properties written to `Output` before the error are kept and returned with the failure.
