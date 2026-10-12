# Artifacts

A `.jyrx` artifact is a compiled script in a binary form that can be stored, transmitted, and loaded later without the source. `JyroCompiler.CompileToArtifact` writes one and `JyroArtifactLoader` loads one.

```csharp
using RealTree.Jyro;

var written = new JyroCompiler()
    .WithStdlib()
    .WithSource("Output.greeting = ToUpper(Input.name)")
    .CompileToArtifact();

if (!written.TryGetValue(out byte[] artifact))
    return;

File.WriteAllBytes("greeting.jyrx", artifact);

CompiledProgram program = new JyroArtifactLoader(File.ReadAllBytes("greeting.jyrx"))
    .WithStdlib()
    .Load()
    .GetValueOrThrow();

var result = new JyroRunner(program)
    .WithInputJson("""{ "name": "ada" }""")
    .Execute(); // {"greeting":"ADA"}
```

## JyroArtifactLoader

| Member | Effect |
| --- | --- |
| `JyroArtifactLoader(byte[])` | The artifact to load. |
| `WithStdlib()` | Makes the standard library available. |
| `AddFunction(IJyroFunction)`, `AddFunctions(IEnumerable<IJyroFunction>)` | Registers host functions. |
| `WithMaxEffect(FunctionEffect)` | The highest effect the program may call. |
| `WithMaxCompilationTime(TimeSpan)` | The time limit for the load. |
| `WithStats(JyroPipelineStats)` | Records the time of each stage. |
| `Load()` | Returns `JyroResult<CompiledProgram>`. |

A load runs the Deserialize, Validate, Link, and Compile stages. The artifact carries the program the parser produced, and the validator checks it as it checks source. The loading host supplies the functions the artifact names and sets its own effect ceiling; the artifact records neither. A function the artifact requires that the host does not supply is JM4100 `UndefinedFunction`, and a function whose effect is above the loader's ceiling is JM7004 `ArtifactEffectExceedsMax`. A refusal is reported in the result, never thrown.

## JyroArtifactInfo

`JyroArtifactInfo.Read(byte[])` and `Read(ReadOnlySpan<byte>)` read the header and the function table without decoding the program.

| Member | Content |
| --- | --- |
| `FormatVersion` | The format version, 1 |
| `SourceHash` | The 32 bytes of the SHA-256 of the UTF-8 source the artifact was compiled from |
| `RequiredFunctions` | The names of the host and standard library functions the program calls |

```csharp
using RealTree.Jyro;

byte[] artifact = File.ReadAllBytes("greeting.jyrx");
if (JyroArtifactInfo.Read(artifact).TryGetValue(out JyroArtifactInfo info))
{
    Console.WriteLine(info.FormatVersion);
    Console.WriteLine(Convert.ToHexString(info.SourceHash.Span));
    Console.WriteLine(string.Join(", ", info.RequiredFunctions));
}
```

`JyroArtifactInfo` has value equality, and `ToString()` gives the version, the hash as 64 lowercase hexadecimal digits, and the required functions on one line. `RequiredFunctions` equals the `RequiredFunctions` of the `CompiledProgram` the artifact was written from, and of the program a load gives. The read applies the same size, version, and table guards as a load.

## Format

An artifact is written little-endian in this order.

| Field | Size |
| --- | --- |
| Magic `JYRX` | 4 bytes, ASCII |
| Format version, 1 | 16-bit unsigned integer |
| Flags, 0 | 16-bit unsigned integer, reserved |
| SHA-256 of the source | 32 bytes |
| Function count | 32-bit unsigned integer |
| Function names | Each a 16-bit length followed by UTF-8 bytes |
| The program | The validated syntax tree |

The function table lists the host and standard library functions the program calls. The script's own functions and variant constructors are part of the program and are not listed.

| Limit | Value |
| --- | --- |
| Artifact size | 10 MB |
| String length | 65,535 bytes |
| List length | 10,000 |
| Function table entries | 1,000 |
| Nesting depth | 256 |

## Versioning

Only format version 1 is accepted. Any other version is refused with JM7002 `ArtifactVersionUnsupported`, and the remedy is to compile the source again. The version changes only when the encoding changes incompatibly.

| Condition | Diagnostic |
| --- | --- |
| The artifact is larger than 10 MB | JM7000 `ArtifactTooLarge` |
| The magic is not `JYRX` | JM7001 `ArtifactInvalidMagic` |
| The version is not 1 | JM7002 `ArtifactVersionUnsupported` |
| The data is truncated, holds invalid UTF-8, repeats a name, exceeds a limit, or has trailing bytes | JM7003 `ArtifactUnreadable` |
| A function's effect is above the loader's ceiling | JM7004 `ArtifactEffectExceedsMax` |
| The compilation has a library | JM7005 `ArtifactCannotEncodeLibraries` |
| The program cannot be encoded or written | JM7007 `ArtifactCannotEncode`, JM7008 `ArtifactUnwritable` |

The header carries a hash of the source, not a signature. An artifact is re-validated when it is loaded, and a host that needs to authenticate an artifact signs it itself.
