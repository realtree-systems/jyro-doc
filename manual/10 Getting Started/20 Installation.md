# Installation

Jyro is available as a command line tool for Windows, macOS, and Linux, as a NuGet package for .NET applications, and as a Visual Studio Code extension.

## Command line tool

The `jyro` command line tool compiles, runs, validates, and tests scripts, and shows this manual with `jyro man`. The tool is self-contained and does not require a .NET runtime to be installed.

### Windows

[Placeholder: download link for the MSI installer]

The installer places the tool in `Program Files\RealTree\Jyro` and adds that directory to the system `PATH`.

### macOS

[Placeholder: Homebrew tap and download link]

### Linux

[Placeholder: download link for the tar.gz archive]

Extract the archive to a directory on the `PATH`.

### Verifying the installation

```bash
jyro --version
```

The command prints the installed version.

## .NET package

The `RealTree.Jyro` package contains the parser, the compiler, the standard library, and the host API. It targets .NET 8.0 and has one dependency, FParsec.

```bash
dotnet add package RealTree.Jyro
```

All public types are in the `RealTree.Jyro` namespace.

## Visual Studio Code extension

[Placeholder: marketplace link]

The extension provides syntax highlighting and diagnostics for `.jyro` files.
