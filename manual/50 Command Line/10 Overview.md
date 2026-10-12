# Command Line Overview

The `jyro` tool compiles, runs, validates, and tests scripts from a terminal, and shows this manual.

```bash
jyro <command> [options]
```

| Command | Alias | Purpose |
| --- | --- | --- |
| [run](/docs/command-line/run) | `r` | Run a `.jyro` script or a `.jyrx` artifact |
| [compile](/docs/command-line/compile) | `c` | Compile a `.jyro` script to a `.jyrx` artifact |
| [validate](/docs/command-line/validate) | `v` | Verify that a `.jyrx` artifact was compiled from a given `.jyro` source |
| [test](/docs/command-line/test) | `t` | Run a script and compare its output with expected JSON |
| [man](/docs/command-line/man) | | Browse the manual |
| [init-config](/docs/command-line/init-config) | | Write a default `jyro.config.json` |

`jyro --version` prints the version. `jyro --help`, `-h`, or `-?` prints the usage of the tool, and `jyro <command> --help` the usage of a command. A bare `jyro` prints the usage.

## Exit codes

| Code | Meaning |
| --- | --- |
| 0 | The command succeeded; usage was printed; `man` showed a page or the index |
| 1 | A usage error; a file was not found or could not be read; the script failed to compile or run; `validate` reported a mismatch; `test` reported a difference; `man` found no page or several; `init-config` refused to overwrite |

No other exit code is used.

## Logging options

Every command except `man` and `init-config` accepts these options.

| Option | Alias | Default | Effect |
| --- | --- | --- | --- |
| `--log-file <path>` | `-l` | none | Writes log output to the file as well as the console |
| `--verbosity <level>` | `-v` | `Information` | The minimum level: `Trace`, `Debug`, `Information`, `Warning`, `Error`, `Critical`, or `None` |
| `--quiet` | `-q` | off | Disables all log output |
| `--console-logging <true\|false>` | | `true` | Enables or disables log output to the console |
| `--log-format <text\|json>` | | `text` | The log format; `json` writes one JSON object per line |

At the top level `v` is the alias of `validate`; inside a command `-v` is the verbosity.

## Configuration

The `run` command reads its settings from three sources, each overriding the one before it: a `jyro.config.json` file, environment variables with the prefix `JYRO_`, and the command line. The other commands take their settings from the command line only.

The configuration file is the one named with `--config`, or else the first found of:

1. `jyro.config.json` in the current directory;
2. `jyro.config.json` in the `Jyro` folder of the user's application data directory;
3. `jyro.config.json` in the user's home directory.

The fields of the file are listed under [init-config](/docs/command-line/init-config).

## Script files

A `.jyro` file holds Jyro source as UTF-8 text. A `.jyrx` file holds a compiled artifact, which embeds the SHA-256 of the source it was compiled from; see [Artifacts](/docs/host-api/compiling/artifacts). Input and output files are JSON.
