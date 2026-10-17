# init-config

Writes a `jyro.config.json` file holding every setting of the `run` command with its default value.

```bash
jyro init-config [-o <path>] [--global] [-f]
```

| Option | Alias | Effect |
| --- | --- | --- |
| `--output <path>` | `-o` | The file to write. Defaults to `jyro.config.json` in the current directory. Overrides `--global`. |
| `--global` | | Writes to the per-user location: the `Jyro` folder of the user's application data directory. |
| `--force` | `-f` | Overwrites an existing file. Without it, an existing file is left unchanged and the exit code is 1. |

## The configuration file

```json
{
  "inputScriptFile": "",
  "inputJsonFile": "",
  "parametersSource": "",
  "outputJsonFile": "",
  "logFile": "",
  "logLevel": "Information",
  "noLogging": false,
  "consoleLogging": false,
  "logFormat": "text",
  "stats": false,
  "pluginAssemblies": "",
  "pluginDirectories": "",
  "pluginRecursiveDirectories": ""
}
```

| Field | Corresponding option |
| --- | --- |
| `inputScriptFile` | `-s` |
| `inputJsonFile` | `-i` |
| `parametersSource` | `-p` |
| `outputJsonFile` | `-o` |
| `logFile` | `--log-file` |
| `logLevel` | `--verbosity` |
| `noLogging` | `--quiet` |
| `consoleLogging` | `--console-logging` |
| `logFormat` | `--log-format` |
| `stats` | `--stats` |
| `pluginAssemblies` | `--plugin` |
| `pluginDirectories` | `--plugin-directory` |
| `pluginRecursiveDirectories` | `--plugin-recursive` |

An empty string means the setting is not set. A setting may also be given as an environment variable with the prefix `JYRO_`, for example `JYRO_logLevel=Debug`. The command line overrides the environment, which overrides the file. Only the `run` command reads the file; see [Overview](/docs/command-line/overview) for the search order.
