# Introduction

Jyro is a scripting language for .NET applications. A script is compiled by the host application and run against JSON data that the host supplies. The script reads that data through `Input`, writes its result to `Output`, and has no access to the file system, the network, the clock, or reflection beyond the functions the host chooses to expose. Every run is bounded by resource limits on time, statement count, loop iterations, and call depth.

Values are the JSON types: null, boolean, number, string, array, and object, plus function values made by lambda expressions. Numbers are exact decimals wherever the value fits; IEEE 754 doubles are used only for values that do not.

A script is compiled once to a .NET delegate and run many times with different data. The compiled form can also be stored as a `.jyrx` artifact and loaded later.

## This manual

| Section | What it covers |
| --- | --- |
| Getting started | Installing Jyro and running a first script |
| Language | The syntax and semantics of Jyro |
| Standard library | The functions every script can call |
| Host API | Embedding Jyro in a .NET application |
| Command line | The `jyro` tool |

## Conventions

Jyro source is shown in fenced blocks. Where a block is followed by a JSON block, that JSON is the `Output` the script produces. Where it is followed by a text block, that text is the diagnostic the script produces.

```jyro
Output.total = 2 + 3
```

```json
{ "total": 5 }
```

A function signature is shown with its parameter names. A parameter in square brackets is optional.

```jyro
# fragment
Substring(text, start, [length])
```

A diagnostic is identified by its code and name, for example JM5200 `DivisionByZero`. A diagnostic is rendered as `[JM5200] Ln 3, Col 15: Division by zero`, where the line and column are one-based positions in the source.

Type names in signatures and tables are written in lowercase: `number`, `string`, `boolean`, `array`, `object`, `function`, `null`, and `any`.
