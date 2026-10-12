# JyroFunction

A `JyroFunction` is a function value. A script makes one with a lambda expression and passes it to a standard library or host function, which calls it. A host makes one to pass a callable into a script, or receives one as an argument.

| Member | Description |
| --- | --- |
| `JyroFunction(Func<IReadOnlyList<JyroValue>, JyroValue> body, int parameterCount)` | A function value over a .NET delegate |
| `ParameterCount` | The number of parameters the function declares |
| `Invoke(IReadOnlyList<JyroValue>)` | Calls the function with the arguments given |

A host function that takes a lambda reads it with `args.Get<JyroFunction>(index)`, checks `ParameterCount`, and calls `Invoke`. A call made this way counts one level of call depth and is bounded by the run's limits.

```csharp
using RealTree.Jyro;

public sealed class ApplyTwiceFunction : JyroFunctionBase
{
    public ApplyTwiceFunction()
        : base("ApplyTwice", JyroFunctionSignature.Create(ParameterType.Any,
            Parameter.Required("value", ParameterType.Any),
            Parameter.Required("callback", ParameterType.Function)))
    {
    }

    public override FunctionEffect Effect => FunctionEffect.Pure;

    public override JyroValue Execute(IReadOnlyList<JyroValue> args, JyroExecutionContext ctx)
    {
        var callback = args.Get<JyroFunction>(1);
        if (callback.ParameterCount != 1)
            throw JyroError.Runtime(MessageCode.CallbackArityMismatch, "ApplyTwice", 1, callback.ParameterCount);
        var once = callback.Invoke(new[] { args[0] });
        return callback.Invoke(new[] { once });
    }
}
```

```jyro
# fragment
Output.a = ApplyTwice(3, x => x * 2)   # 12
```

Function values compare by reference: each evaluation of a lambda expression makes a new value, and `EqualsValue` is `true` only for the same value. A function value has no JSON form; `ToJson` writes it as a string, and `ToObjectValue` gives `"<function>"`. A function value cannot be returned from a script as part of `Output` in a form a host can use beyond that.
