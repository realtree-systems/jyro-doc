# Registration

Host functions are registered on the compiler, or on the loader, with `AddFunction(IJyroFunction)` or `AddFunctions(IEnumerable<IJyroFunction>)`. They are resolved at link time and baked into the program, so a runner needs no function set. `AddFunctions` enumerates its sequence once.

```csharp
using RealTree.Jyro;

CompiledProgram program = new JyroCompiler()
    .WithStdlib()
    .AddFunction(new ReverseStringFunction())
    .AddFunctions(new IJyroFunction[] { new WithdrawFunction(), new SumAllFunction() })
    .WithSource("Output.r = ReverseString('abc')")
    .CompileOrThrow();
```

## Names

| Condition | Diagnostic |
| --- | --- |
| A host function with the name of a standard library function | JM4105 `HostFunctionOverridesStdlib` |
| Two host functions with the same name | JM4101 `DuplicateFunction` |
| A script `func` with the name of a host function | JM4102 `FunctionOverride` |
| A script variant with the name of a host function | JM4300 `VariantShadowsFunction` |

There is no override of a standard library function. The check against the standard library's names applies whether or not `WithStdlib` was called.

## The standard library

`WithStdlib()` registers the standard library. Without it, no standard library function can be called, and a script that calls one fails at link time with JM4100 `UndefinedFunction`. `StdlibRegistry.GetAll()` enumerates the library as an `IReadOnlyList<IJyroFunction>`, and `StdlibRegistry.TryGetFunction(name, out function)` finds one function by name, giving the same instance for a name on every call. A host that wants a subset of the library registers the functions it chooses with `AddFunctions` instead of calling `WithStdlib`.

```csharp
using RealTree.Jyro;

IEnumerable<IJyroFunction> allowed = StdlibRegistry.GetAll()
    .Where(f => f.Effect == FunctionEffect.Pure);

CompiledProgram program = new JyroCompiler()
    .AddFunctions(allowed)
    .WithSource("Output.n = Length(Input)")
    .CompileOrThrow();
```

## Artifacts

An artifact names the functions it requires. The loading host registers them with the same members on `JyroArtifactLoader`; `JyroArtifactInfo.RequiredFunctions` lists the names without loading the program, and `CompiledProgram.RequiredFunctions` lists the same names for a compiled or loaded program. See [Artifacts](/docs/host-api/compiling/artifacts).
