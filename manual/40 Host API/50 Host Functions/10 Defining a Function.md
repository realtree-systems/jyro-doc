# Defining a Function

A host function is a .NET class that a script calls by name. It is derived from `JyroFunctionBase` and defines four things: a name, a signature, an effect, and an implementation.

```csharp
using RealTree.Jyro;

public sealed class ReverseStringFunction : JyroFunctionBase
{
    public ReverseStringFunction()
        : base("ReverseString",
               JyroFunctionSignature.Create(ParameterType.String,
                   Parameter.Required("text", ParameterType.String)))
    {
    }

    public override FunctionEffect Effect => FunctionEffect.Pure;

    public override JyroValue Execute(IReadOnlyList<JyroValue> args, JyroExecutionContext ctx)
    {
        char[] chars = args.GetString(0).ToCharArray();
        Array.Reverse(chars);
        return new JyroString(new string(chars));
    }
}
```

```csharp
using RealTree.Jyro;

CompiledProgram program = new JyroCompiler()
    .WithStdlib()
    .AddFunction(new ReverseStringFunction())
    .WithSource("Output.r = ReverseString(Input.text)")
    .CompileOrThrow();

var result = new JyroRunner(program)
    .WithInputJson("""{ "text": "hello" }""")
    .Execute(); // {"r":"olleh"}
```

A class that cannot derive from `JyroFunctionBase` implements `IJyroFunction` directly, with the members `Name`, `Signature`, `Effect`, and `Execute`.

## Name

The name is the identifier a script calls. It is case-sensitive and must not be the name of a standard library function; see [Registration](/docs/host-api/host-functions/registration).

## Signature

`JyroFunctionSignature.Create(returnType, params Parameter[])` or `Create(IEnumerable<Parameter>, returnType)` declares the parameters in order and the return type. `Parameter.Required(name, type)` and `Parameter.Optional(name, type)` make the parameters; every optional parameter follows every required one. A signature is immutable and keeps its own copy of the parameters.

| Member | Content |
| --- | --- |
| `Parameters` | The parameters, each with `Name`, `Type`, and `IsOptional` |
| `ReturnType` | The declared return type |
| `MinArgs` | The number of required parameters |
| `MaxArgs` | The number of parameters |

Parameter names are part of the call surface: a script passes arguments by name with them. The compiler checks the argument count and the names at link time. `ParameterType` is `Null`, `Boolean`, `Number`, `String`, `Array`, `Object`, `Function`, or `Any`. Declared types are not enforced by the compiler or the runtime; the function checks its arguments when it reads them.

## Effect

`Effect` is abstract. It declares what a call may do beyond computing a result: `Pure`, `NonDeterministic`, `Blocking`, or `Mutating`. See [Effects](/docs/host-api/host-functions/effects).

## Execute

`Execute(IReadOnlyList<JyroValue> args, JyroExecutionContext ctx)` receives the arguments in parameter order and returns the result as a `JyroValue`. An omitted optional argument, and an explicit `null`, arrive as `JyroNull`. The result is any `JyroValue`; `JyroNull.Instance` is the Jyro `null`.

### Reading arguments

The extension methods of `JyroArguments` read an argument by index. An index past the end reads as `null`.

| Reader | Returns | Rule |
| --- | --- | --- |
| `GetString(i)` | `string` | A number or boolean as its text; `null` as `""`; an array, object, or function refused |
| `GetBoolean(i)` | `bool` | A boolean; anything else, `null` included, refused |
| `GetNumber(i)` | `double` | A number as a double; lossy above 2^53 and for high-precision decimals |
| `GetDecimal(i)` | `decimal` | A number, exact |
| `GetInt64(i)`, `GetInt32(i)` | `long`, `int` | A whole number in range; otherwise JM5904 `IntegerRequired` or JM5911 `IntegerOutOfRange` |
| `GetNonNegativeInt32(i)` | `int` | As `GetInt32`, and refuses a negative value with JM5907 |
| `GetArray(i)`, `GetObject(i)` | `JyroArray`, `JyroObject` | `null` as the empty container; another type refused |
| `Get<T>(i)` | `T` | The argument as the subclass `T`, or refused; `null` is a `T` only for `JyroValue` and `JyroNull` |
| `GetOptional<T>(i, T fallback)` | `T` | `fallback` when the argument is absent or `null`; a wrong type is still refused |

A refusal is a `JyroRuntimeException` with JM5104 `ArgumentTypeMismatch`, whose arguments are the expected and actual type names as `TypeOf` spells them. During a run the diagnostic names the function and the parameter.

### The execution context

| Member | Content |
| --- | --- |
| `Input`, `Output` | The run's values, read-only |
| `HostState` | The object given to `JyroRunner.WithHostState`, or `null` |
| `CancellationToken` | Fires at the execution time limit or when the host cancels |
| `HostCancellationRequested` | Whether the host's own token was canceled |
| `CheckExecutionLimits()` | Raises the time limit or cancellation error when either applies |
| `CallDepth`, `PeakCallDepth` | The current and the greatest call depth of the run |

A function that blocks passes `ctx.CancellationToken` to the operation, so the time limit and the host can stop it. A function that iterates calls `ctx.CheckExecutionLimits()` as it goes. The hooks that enforce the limits are internal; a function cannot change the limits of the run it is called in.

```csharp
using RealTree.Jyro;

public sealed class SumAllFunction : JyroFunctionBase
{
    public SumAllFunction()
        : base("SumAll", JyroFunctionSignature.Create(ParameterType.Number,
            Parameter.Required("values", ParameterType.Array),
            Parameter.Optional("start", ParameterType.Number)))
    {
    }

    public override FunctionEffect Effect => FunctionEffect.Pure;

    public override JyroValue Execute(IReadOnlyList<JyroValue> args, JyroExecutionContext ctx)
    {
        decimal total = args.GetOptional(1, new JyroNumber(0)).DecimalValue;
        foreach (JyroValue value in args.GetArray(0))
        {
            ctx.CheckExecutionLimits();
            total += value.AsNumber().DecimalValue;
        }
        return new JyroNumber(total);
    }
}
```

## Testing a function

A function is called outside the pipeline through `IJyroFunction` with a bare context.

```csharp
using RealTree.Jyro;

IJyroFunction reverse = new ReverseStringFunction();
JyroValue value = reverse.Execute(new JyroValue[] { new JyroString("abc") }, new JyroExecutionContext());
Console.WriteLine(value.ToStringValue()); // cba
```

## Thread safety

A function instance is baked into the program and invoked concurrently by every run. It holds no mutable state. Per-run state is passed with `WithHostState` and read as `ctx.HostState`.
