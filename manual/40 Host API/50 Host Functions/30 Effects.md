# Effects

Every function declares a `FunctionEffect`. The compiler compares the effect of each call against a ceiling the host sets, and refuses a script that calls beyond it.

| Effect | Value | Meaning |
| --- | --- | --- |
| `Pure` | 0 | The result depends only on the arguments |
| `NonDeterministic` | 1 | The function reads the clock or a random source |
| `Blocking` | 2 | The function may pause execution, for example on I/O |
| `Mutating` | 3 | The function reads or writes state outside the script |

The values are ordered, so `FunctionEffect.Blocking < FunctionEffect.Mutating`. A host function declares its true effect. A function with an effect weakens the sandbox in proportion to what it can do; it takes the narrowest arguments it can work with and checks them.

## Ceilings

| Ceiling | Covers | Set with | Default |
| --- | --- | --- | --- |
| Script | Calls in the script's statements and functions | `JyroCompiler.WithMaxEffect` | `Mutating` |
| Library | Calls in the bodies of attached libraries | `JyroCompiler.WithLibraryMaxEffect` | `NonDeterministic` |
| Artifact | The functions an artifact names | `JyroArtifactLoader.WithMaxEffect` | `Mutating` |

A call above the ceiling fails at link time with JM4104 `FunctionEffectExceedsMax`, whose arguments are the function, its effect, and the ceiling. An artifact that names a function above the loader's ceiling fails with JM7004 `ArtifactEffectExceedsMax`.

```csharp
using RealTree.Jyro;

var result = new JyroCompiler()
    .WithStdlib()
    .WithMaxEffect(FunctionEffect.Pure)
    .WithSource("Output.id = NewGuid()")
    .Compile();

// result.IsSuccess is false: NewGuid is NonDeterministic.
// [JM4104] Ln 1, Col 13: Call to 'NewGuid' with effect NonDeterministic exceeds the maximum effect Pure
```

The ceiling is applied when the program is linked and is not recorded in the `CompiledProgram`. A host that serves tenants under different policies compiles one program per policy; resource limits, by contrast, are set per run.

## User functions

The effect of a function declared with `func` is the highest effect of the functions its body calls. The ceiling applies to the call sites in the body, so a script function that calls `Sleep` is refused under a `Blocking` or lower ceiling at the `Sleep` call.

## Effect that cannot be read

An `Effect` getter that throws, or that returns a value other than the four members, fails the compilation with JM4106 `MaxEffectNotApplicable`, at each call of the function in a source, or for each such function in an artifact's table.
