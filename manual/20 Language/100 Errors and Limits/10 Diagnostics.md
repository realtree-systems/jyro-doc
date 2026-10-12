# Diagnostics

Every error, whether found while the script is compiled or while it runs, is reported as a diagnostic. A diagnostic carries:

| Part | Content |
| --- | --- |
| Code | A number in the form `JMxxxx` and a name, such as JM5200 `DivisionByZero` |
| Severity | Info, Warning, or Error. The language itself reports only errors. |
| Stage | The stage that produced the diagnostic |
| Message | The text, rendered from the code's template and the diagnostic's arguments |
| Arguments | The values substituted into the template, such as a name or a type |
| Location | The line, column, and length of the source the diagnostic refers to, or none |

Lines and columns are one-based and count UTF-16 code units. A diagnostic is rendered as `[JMxxxx] Ln L, Col C: message`, or as `[JMxxxx]: message` when it has no location.

```jyro
var count = 0
Output.ratio = 10 / count
```

```text
[JM5200] Ln 2, Col 16: Division by zero
```

## Stages

| Stage | Reports | Collection |
| --- | --- | --- |
| Parse | Lexical and syntax errors: unterminated strings, bad escapes, malformed numbers, unexpected tokens | Every lexical error, and the first syntax error |
| Validate | Static rules: undeclared variables, misplaced declarations, invalid targets, match exhaustiveness, reserved names | Every error found |
| Link | Name resolution and call checks: unknown functions, argument counts and names, effect ceilings, clashes with library and host functions | Every error found |
| Compile | Internal failures of the compiler, and the compilation time limit | |
| Execute | Runtime errors, `fail`, resource limits, and host cancellation | The first error; the run stops |
| Deserialize | A `.jyrx` artifact that cannot be read | |

A script with a compilation error does not run. A script that runs is one that passed every earlier stage.

## Code bands

The first digit of a code gives the part of the system that defines it.

| Band | Part |
| --- | --- |
| JM1xxx | Lexer |
| JM2xxx | Parser |
| JM3xxx | Validator |
| JM4xxx | Linker |
| JM5xxx | Runtime |
| JM6xxx | Compiler |
| JM7xxx | Artifact reader and writer |

The full list of codes and their message templates is in [Error Codes](/docs/language/errors-and-limits/error-codes). How a host reads diagnostics is described in [Diagnostics](/docs/host-api/diagnostics) in the Host API section.
