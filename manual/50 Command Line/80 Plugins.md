# Plugins

A plugin is a .NET assembly that contains host functions. The `run`, `compile`, and `test` commands load plugins and make their functions callable from the script.

| Option | Alias | Effect |
| --- | --- | --- |
| `--plugin <paths>` | | Comma-separated paths of assemblies. An assembly that cannot be loaded is an error. |
| `--plugin-directory <dirs>` | `-pd` | Comma-separated directories. Every `.dll` in each directory, top level only, is loaded; a file that cannot be loaded is skipped. |
| `--plugin-recursive <dirs>` | `-pr` | As `--plugin-directory`, searching subdirectories as well. |

Every concrete type in a loaded assembly that implements `IJyroFunction` is instantiated through its public parameterless constructor and registered. A type without such a constructor is an error for an assembly named with `--plugin`, and is skipped for one found in a directory. The rules for writing a function are given in [Defining a Function](/docs/host-api/host-functions/defining-a-function); a plugin references the `RealTree.Jyro` package.

```bash
jyro run -s report.jyro -i orders.json --plugin ./plugins/Acme.Jyro.dll
```

A plugin function with the name of a standard library function, or two plugin functions with the same name, fail the compilation as described in [Registration](/docs/host-api/host-functions/registration).

## WriteLog

The tool adds one host function of its own to every script.

```jyro
# fragment
WriteLog(level, message)
```

| Parameter | Type | Description |
| --- | --- | --- |
| `level` | string | `trace`, `debug`, `info` or `information`, `warn` or `warning`, `error`, or `critical`, in any case. Any other value is a runtime error. |
| `message` | string | The text to log. |

The message is written to the tool's log, prefixed `[Script]`, at the given level, and is subject to the logging options. The function returns `null` and has the effect NonDeterministic.

```jyro
# fragment
WriteLog("info", "Processing " & Length(Input.orders) & " orders")
```
