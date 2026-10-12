# compile

Compiles a `.jyro` script to a `.jyrx` artifact.

```bash
jyro compile -s <script.jyro> [-o <output.jyrx>] [options]
```

| Option | Alias | Effect |
| --- | --- | --- |
| `--input-script-file <path>` | `-s` | The script to compile. Required. |
| `--output-file <path>` | `-o` | The artifact to write. Defaults to the script's path with the extension `.jyrx`. |
| `--plugin <paths>`, `--plugin-directory <dirs>`, `--plugin-recursive <dirs>` | , `-pd`, `-pr` | Plugin assemblies whose functions the script may call; see [Plugins](/docs/command-line/plugins). |

The logging options are listed in the [Overview](/docs/command-line/overview). The command does not read the configuration file.

The script is compiled with the standard library, the `WriteLog` function, and the plugin functions. On success the command prints the paths and the size of the artifact:

```text
Compiled report.jyro -> report.jyrx (412 bytes)
```

On failure the diagnostics are printed to standard error and the exit code is 1. A script that uses a library cannot be compiled to an artifact.

The artifact records the names of the host and standard library functions the script calls, and the SHA-256 of the source. A host that loads it supplies those functions; the `run` command supplies the standard library, `WriteLog`, and the plugins given on its own command line. See [Artifacts](/docs/host-api/compiling/artifacts) for the format.
