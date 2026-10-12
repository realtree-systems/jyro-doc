# Raising Errors

A host function reports a failure by throwing an exception made by `JyroError`. The run ends with a failed result whose value is `Output` as it stood.

## JyroError.Host

`JyroError.Host(string reason, params object[] values)` returns a `JyroRuntimeException` for a condition the function rejects on purpose: a value outside its range, an unavailable resource, a business rule. The run fails with JM5006 `HostFunctionError`, located at the call. The diagnostic's arguments are the reason followed by the values.

```csharp
using RealTree.Jyro;

public sealed class WithdrawFunction : JyroFunctionBase
{
    public WithdrawFunction()
        : base("Withdraw", JyroFunctionSignature.Create(ParameterType.Number,
            Parameter.Required("amount", ParameterType.Number)))
    {
    }

    public override FunctionEffect Effect => FunctionEffect.Pure;

    public override JyroValue Execute(IReadOnlyList<JyroValue> args, JyroExecutionContext ctx)
    {
        decimal amount = args.GetDecimal(0);
        if (amount <= 0)
            throw JyroError.Host("AmountNotPositive", amount);
        return new JyroNumber(amount);
    }
}
```

The reason is a name the host can switch on, not a sentence. A sentence, or a translated one, is produced from the diagnostic's `Args`, or by giving `HostFunctionError` a template through `IMessageTemplateProvider`; see [Diagnostics](/docs/host-api/diagnostics). With neither, the default message names the reason and the first nine values.

## JyroError.Runtime

`JyroError.Runtime(MessageCode code, params object[] args)` returns an exception with one of the argument and conversion codes the standard library uses, such as JM5104 `ArgumentTypeMismatch` or JM5917 `InvalidJson`, and the run fails with that code. A code that only the engine raises is refused with `ArgumentOutOfRangeException`: every code below 5000 or from 6000 up, and `RuntimeError`, `CanceledByHost`, `ScriptFailure`, `HostFunctionError`, the limit codes JM5900 to JM5903, `ValueNestingLimitExceeded`, and `ValueSizeLimitExceeded`.

## Other exceptions

Any other exception that escapes `Execute` is reported as JM5002 `RuntimeError`, located at the call. Its one argument is the function's name followed by `()`. The exception's type and message are not carried, so a condition the function meant to reject is indistinguishable from a fault.

| Thrown | Reported as |
| --- | --- |
| `JyroError.Host(reason, values)` | JM5006 `HostFunctionError` with the reason and values |
| `JyroError.Runtime(code, args)` | The code given, with its arguments |
| An `ArgumentTypeMismatch` from a `JyroArguments` reader | JM5104 naming the function and the parameter |
| `OperationCanceledException` from `ctx.CancellationToken` | JM5903 `ExecutionTimeLimitExceeded` or JM5003 `CanceledByHost` |
| Any other exception | JM5002 `RuntimeError` naming the function |

## Errors at compile time

An exception made by `JyroError.Host` and thrown from `CheckCall` refuses the call with JM4001 `HostLinkError`; see [Call Checks](/docs/host-api/host-functions/call-checks). An exception thrown by `Name`, `Signature`, or `Effect` during compilation is reported in the result of `Compile`, `CompileToArtifact`, or `Load` as JM6000 `UnknownCompilerError`, or JM4106 `MaxEffectNotApplicable` for `Effect`.
