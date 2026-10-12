# Call Checks

A host function may refuse a call when the script is compiled, before any script runs. `JyroFunctionBase.CheckCall(JyroCallSite)` is called once for each call of the function in the script and in each library, after the call has passed the checks of name, argument count, named arguments, and effect. Loading an artifact calls it the same way. The default implementation accepts every call. A class that does not derive from `JyroFunctionBase` implements `IJyroCallCheck`.

| `JyroCallSite` member | Content |
| --- | --- |
| `Name` | The function's name |
| `ArgumentCount` | The number of arguments at the call |
| `ArgumentNames` | The names of a call with named arguments, in call order; empty for positional arguments |
| `TryGetLiteral(int index, out JyroValue value)` | The argument's value when it is written as a literal; `false` for any other expression |

```csharp
using RealTree.Jyro;

public sealed class QueryFunction : JyroFunctionBase
{
    private static readonly HashSet<string> Tables = new() { "orders", "customers" };

    public QueryFunction()
        : base("Query", JyroFunctionSignature.Create(ParameterType.Array,
            Parameter.Required("table", ParameterType.String),
            Parameter.Required("filter", ParameterType.Object)))
    {
    }

    public override FunctionEffect Effect => FunctionEffect.Blocking;

    public override void CheckCall(JyroCallSite call)
    {
        if (call.TryGetLiteral(0, out var table) && table is JyroString name && !Tables.Contains(name.Value))
            throw JyroError.Host("TableUnknown", name.Value);
    }

    public override JyroValue Execute(IReadOnlyList<JyroValue> args, JyroExecutionContext ctx)
    {
        return new JyroArray();
    }
}
```

A check returns to accept the call. An exception made by `JyroError.Host` refuses it: the compilation fails with JM4001 `HostLinkError`, located at the call, whose arguments are the reason followed by the values. Each refused call gives its own diagnostic.

| Thrown from `CheckCall` | Reported as |
| --- | --- |
| `JyroError.Host(reason, values)` | JM4001 `HostLinkError` at the call |
| `JyroError.Runtime(code, args)` | The code given, at stage Link |
| Any other exception, including an engine `JyroRuntimeException` such as `InvalidCast` | JM6000 `UnknownCompilerError` at stage Link |

The check sees only what is written at the call. An argument computed at run time is not a literal, and `TryGetLiteral` returns `false` for it; a function that must restrict such arguments checks them in `Execute` as well.
