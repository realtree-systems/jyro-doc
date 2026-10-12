# Jyro Reference Manual

The reference documentation for the Jyro programming language, as published at www.jyro.dev. The manual is markdown under `manual/`, one file per page, in the layout the site reads:

```
manual/<NN Section>/[<NN Subsection>/]<NN Page>.md
```

The number at the start of each name sets the order and is dropped from the title. `20 Language/30 Types and Values/20 Numbers.md` is the page "Numbers" under "Language > Types and Values", served at `/docs/language/types-and-values/numbers`. A page is one H1 followed by GitHub-flavored markdown. Fenced blocks are tagged `jyro`, `csharp`, `json`, `bash`, or `text`.

## Sections

| Section | What it covers |
| --- | --- |
| Getting started | Installing Jyro and running a first script |
| Language | The syntax and semantics of Jyro |
| Standard library | The functions every script can call |
| Host API | Embedding Jyro in a .NET application |
| Command line | The `jyro` tool |

## Generated pages

The standard library pages, the category tables, and the error code table are generated from the Jyro build by `docs/tools/Generate.fsx` in the `jyro` repository. A page whose first line is `<!-- hand-written -->` is never touched by the generator; a page whose first line is `<!-- generated ... -->` is replaced on the next run. The scripts find this repository beside `jyro`, or under the root named by the `JYRO_DOC` environment variable.

```
cd ../jyro
dotnet build -c Release src/Jyro.Api/Jyro.Api.fsproj
dotnet fsi docs/tools/Generate.fsx
```

## Checking the examples

Every `jyro` block is run against the Jyro build and compared with the `json` or `text` block that follows it. Every `csharp` block is compiled against the Release assemblies.

```
cd ../jyro
dotnet fsi docs/tools/CheckExamples.fsx
dotnet fsi docs/tools/CheckCSharp.fsx <scratch-project-dir>
```

A `jyro` block may start with `# fragment`, which skips it, or `# input: <json>`, which sets `Input`. A `csharp` block may start with `// fragment`.

## Publishing

Copy `manual/` over `web/src/content/manual/` in the `www-jyro-dev` repository and build the site. Nothing else needs to change: the site builds its navigation from the file names.
