# Diagnostics

Every failure of compilation, loading, or execution is reported as a `DiagnosticMessage` in the `Messages` of a result, or in the `Messages` of a `JyroException`.

## DiagnosticMessage

| Member | Content |
| --- | --- |
| `Code` | A `MessageCode`, an enum whose values are the numbers of the codes |
| `Severity` | `MessageSeverity.Info`, `Warning`, or `Error` |
| `Stage` | `MessageStage.Deserialize`, `Parse`, `Validate`, `Link`, `Compile`, or `Execute` |
| `Message` | The text, rendered from the code's template and `Args` |
| `Args` | The values of the message: a name, a type, a limit, a token |
| `Location` | A `SourceLocation?`; no value when the diagnostic has no place in the source |

`SourceLocation` has `Line`, `Column`, and `Length`. Lines and columns are one-based and count UTF-16 code units; a CRLF is one line end. `Length` is the number of code units of the place. `ToString()` gives `Ln L, Col C`.

| Diagnostic | Place |
| --- | --- |
| A syntax error | The token at which the parser stopped; at the end of the source, an empty range after the last code unit |
| A lexical error | The token |
| A validation or link error | The node the rule rejects |
| A runtime error | The node whose evaluation raised it: the call, the operator, the access, or the statement |

The band of a code names the part of the system that defines it: JM1xxx lexer, JM2xxx parser, JM3xxx validator, JM4xxx linker, JM5xxx runtime, JM6xxx compiler, JM7xxx artifact. `Stage` names the stage that produced the diagnostic, which differs where a code is raised outside its band: an unpaired surrogate in a literal is a parse-stage diagnostic with a runtime-band code, and every diagnostic of a run has stage Execute. The engine itself reports only errors; `IsSuccess` is `true` when no message has severity Error. The full code list is in [Error Codes](/docs/language/errors-and-limits/error-codes).

## Formatting

| Member | Output |
| --- | --- |
| `DiagnosticFormatter.FormatMessage(message)`, `message.ToString()` | `[JM5200] Ln 12, Col 5: Division by zero`, or `[JM6001]: No source provided` without a location |
| `DiagnosticFormatter.FormatCode(code)` | `JM5200` |
| `DiagnosticFormatter.ToStructured(message)` | A `StructuredDiagnostic` with `Code` as text, `NumericCode`, `Severity`, `Stage`, `Subsystem` (the band name), `Message`, `Args`, and `Location`, suitable for JSON serialization |
| `DiagnosticFormatter.FormatLocalized(provider, message)` | The message rendered with the host's template |

```csharp
using RealTree.Jyro;

var result = new JyroCompiler()
    .WithStdlib()
    .WithSource("Output.a = Length(1, 2)")
    .Compile();

foreach (DiagnosticMessage message in result.Messages)
{
    Console.Error.WriteLine(DiagnosticFormatter.FormatMessage(message));
    StructuredDiagnostic structured = DiagnosticFormatter.ToStructured(message);
    Console.Error.WriteLine($"{structured.Code} {structured.Stage} {structured.Subsystem}");
}
// [JM4201] Ln 1, Col 12: Function 'Length' accepts at most 1 arguments, but 2 were provided
// JM4201 Link linker
```

## Localization

`Message` is derived from the code's English template and `Args`; it is not stored. A host supplies templates in another language by implementing `IMessageTemplateProvider`, whose `TryGetTemplate(MessageCode, out string)` returns a template with `{0}`, `{1}` placeholders for the codes it translates and `false` for the rest. `FormatLocalized` applies the host's template, and falls back to the English message. The message of a `ScriptFailure` is the text given to `fail` and is never templated.

```csharp
using RealTree.Jyro;

public sealed class FrenchTemplates : IMessageTemplateProvider
{
    public bool TryGetTemplate(MessageCode code, out string template)
    {
        switch (code)
        {
            case MessageCode.DivisionByZero:
                template = "Division par zéro";
                return true;
            case MessageCode.InvalidType:
                template = "Impossible de donner une valeur de type {0} à une cible de type {1}";
                return true;
            default:
                template = null!;
                return false;
        }
    }
}
```

`MessageTemplates.Get(code)` and `TryGet(code, out template)` give the English template of a code, `Render(template, args)` fills a template, and `Format(code, args)` does both.

## Constructing diagnostics

`DiagnosticMessage` has no public constructor. `DiagnosticMessage.Error`, `Warning`, and `Info` each take the code and the stage, an optional `SourceLocation`, and the arguments. Two diagnostics are equal when their code, severity, stage, location, and the invariant strings of their arguments are equal.

```csharp
using RealTree.Jyro;

var error = DiagnosticMessage.Error(MessageCode.RuntimeError, MessageStage.Execute, "LoadOrder()");
var located = DiagnosticMessage.Error(MessageCode.UndefinedFunction, MessageStage.Link, new SourceLocation(10, 5, 9), "LoadOrder");
var failure = JyroResult<JyroValue>.Failure(error);
```

## Exceptions

| Exception | Thrown by | Content |
| --- | --- | --- |
| `JyroException` | `CompileOrThrow`, `ExecuteOrThrow`, `GetValueOrThrow` | `Messages`, holding at least one error; `Stage` and `Location` of the first error |
| `JyroRuntimeException`, derived from `JyroException` | `JyroValue.FromJson`, `FromObject`, `ToJson`, the `JyroNumber(double)` constructor, and the `JyroArguments` readers; returned by `JyroError.Host` and `JyroError.Runtime` | `Code` and `Args` of one diagnostic |
| `ArgumentNullException`, `ArgumentOutOfRangeException`, `ArgumentException` | A builder method given `null` or an out-of-range limit; `JyroObject.Add` with an existing key; `JyroResult.Success` with an error | The host's own mistake |

A host constructs neither Jyro exception; `JyroError` makes the one a host function throws. One `catch (JyroException)` handles every exception that carries a Jyro code.
