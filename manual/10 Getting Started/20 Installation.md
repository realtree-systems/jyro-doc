# Installation

Jyro is available as a command line tool for Windows, macOS, and Linux, as a NuGet package for .NET applications, and as a Visual Studio Code extension.

## Command line tool

The `jyro` command line tool compiles, runs, validates, and tests scripts, and shows this manual with `jyro man`. The tool is self-contained and does not require a .NET runtime to be installed.

The current release is [1.0.0-rc.1](https://github.com/realtree-systems/jyro-cli/releases/tag/v1.0.0-rc.1). Its packages are:

| Platform | Package |
|----------|---------|
| Windows x64 | [MSI installer](https://github.com/realtree-systems/jyro-cli/releases/download/v1.0.0-rc.1/jyro-win-x64.msi), [zip archive](https://github.com/realtree-systems/jyro-cli/releases/download/v1.0.0-rc.1/jyro-win-x64.zip) |
| Linux x64 | [Debian package](https://github.com/realtree-systems/jyro-cli/releases/download/v1.0.0-rc.1/jyro-linux-x64.deb), [tar archive](https://github.com/realtree-systems/jyro-cli/releases/download/v1.0.0-rc.1/jyro-linux-x64.tar.gz) |
| Linux ARM64 | [Debian package](https://github.com/realtree-systems/jyro-cli/releases/download/v1.0.0-rc.1/jyro-linux-arm64.deb), [tar archive](https://github.com/realtree-systems/jyro-cli/releases/download/v1.0.0-rc.1/jyro-linux-arm64.tar.gz) |
| macOS Apple silicon | Homebrew, or [tar archive](https://github.com/realtree-systems/jyro-cli/releases/download/v1.0.0-rc.1/jyro-macos-arm64.tar.gz) |

The release also carries `SHA256SUMS.txt`, the SHA-256 checksum of every package.

### Windows

The MSI installer places `Jyro.exe` in `Program Files\RealTree\Jyro`, adds that directory to the system `PATH`, and associates `.jyro` and `.jyrx` files with the tool, so that opening one runs it.

The zip archive holds `Jyro.exe` alone. Place it in a directory on the `PATH`. File names on Windows are not case-sensitive, so the command `jyro` runs it.

### macOS

Homebrew installs the command `jyro`:

```bash
brew install realtree-systems/tap/jyro
```

The tar archive holds one executable, named `Jyro`. Place it in a directory on the `PATH`, rename it to `jyro` for a lowercase command, and mark it executable:

```bash
mv Jyro jyro
chmod +x jyro
```

Gatekeeper may block an executable downloaded with a browser. If it does, remove the quarantine attribute:

```bash
xattr -d com.apple.quarantine jyro
```

A Homebrew install is not affected.

### Linux

The Debian package installs the tool under `/usr/lib/jyro` and provides the command `jyro` in `/usr/bin`:

```bash
sudo apt install ./jyro-linux-x64.deb
```

The tar archive holds one executable, named `Jyro`. Place it in a directory on the `PATH`; file names are case-sensitive, so rename it to `jyro` for a lowercase command, and mark it executable:

```bash
mv Jyro jyro
chmod +x jyro
```

### Verifying the installation

```bash
jyro --version
```

The command prints the installed version, for example `jyro 1.0.0+3bab186`. The text after `+` is the commit the release was built from.

## .NET package

The `RealTree.Jyro` package contains the parser, the compiler, the standard library, and the host API. It targets .NET 8.0 and depends on FParsec and FSharp.Core.

```bash
dotnet add package RealTree.Jyro
```

All public types are in the `RealTree.Jyro` namespace.

## Visual Studio Code extension

Install [Jyro for VS Code](https://marketplace.visualstudio.com/items?itemName=RealTree-Systems.jyro-language) from the Visual Studio Code Marketplace.

The extension provides syntax highlighting and diagnostics for `.jyro` files.
