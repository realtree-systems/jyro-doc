# Libraries

A library is Jyro source that contains only `func` and `union` declarations. It is attached to a compilation, and the script calls its functions and constructs its variants by name. The language rules are given in [Libraries](/docs/language/libraries) in the Language section.

## Attaching a library

`AddLibrarySource(string)` parses the library as part of the compilation. `JyroLibrary.Parse(string)` parses a library once, for reuse across many compilers; the result is attached with `AddLibrary`.

```csharp
using RealTree.Jyro;

const string librarySource = """
    func Triple(n)
        return n * 3
    end
    """;

var parsed = JyroLibrary.Parse(librarySource);
if (!parsed.TryGetValue(out JyroLibrary library))
    return;

CompiledProgram program = new JyroCompiler()
    .WithStdlib()
    .WithSource("Output.r = Triple(Input.n)")
    .AddLibrary(library)
    .CompileOrThrow();

var result = new JyroRunner(program)
    .WithInputJson("""{ "n": 2 }""")
    .Execute(); // {"r":6}
```

`JyroLibrary.Parse(string, TimeSpan)` applies the given compilation time limit; the one-argument form applies the default of 5 seconds. A `JyroLibrary` exposes `DeclarationCount`, the number of declarations it holds, and `DeclaredNames`, the names of the functions and unions it declares, each once, in source order.

## What each step checks

`JyroLibrary.Parse` reports the faults a library has on its own: parse errors, a statement that is not a declaration (JM3509 `LibraryStatementForbidden`), and every validation rule that reads only the library.

Each `Compile` the library is added to reports the faults that depend on the other source texts, with the same stage and place as if the library had been added with `AddLibrarySource`:

- a function name that another source text declares or that a registered function has, JM4101 `DuplicateFunction` or JM4102 `FunctionOverride`;
- a union or variant name that another source text declares, JM3608 or JM3601;
- a `match` whose cases name variants the library does not declare, JM3606, JM3604, JM3611, and JM3602;
- every call, and the library effect ceiling.

## Effect ceiling

Calls in library bodies are checked against `WithLibraryMaxEffect`, which defaults to `NonDeterministic`. Calls in the script are checked against `WithMaxEffect`, which defaults to `Mutating`. See [Effects](/docs/host-api/host-functions/effects).

## Artifacts

A compilation with a library cannot be written as a `.jyrx` artifact: `CompileToArtifact` fails with JM7005 `ArtifactCannotEncodeLibraries`, reported once the script and the libraries have compiled without other errors. `JyroArtifactLoader` has no member that attaches a library.
