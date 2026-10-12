# run

Runs a `.jyro` script or a `.jyrx` artifact and prints `Output` as JSON.

```bash
jyro run -s <script> [-i <input.json>] [-p <parameters>] [-o <output.json>] [options]
jyro run --stdin [-i <input.json>] [-p <parameters>] [-o <output.json>] [options]
```

| Option | Alias | Effect |
| --- | --- | --- |
| `--input-script-file <path>` | `-s` | The script to run. A file with the extension `.jyrx`, in any case, is loaded as an artifact; any other file is compiled as source. |
| `--stdin` | | Reads the script source from standard input. Cannot be combined with `-s`. One of the two is required. |
| `--input-json-file <path>` | `-i` | A JSON file whose contents become `Input`. Without it, `Input` is `null`. |
| `--parameters <json\|path>` | `-p` | Inline JSON when the value starts with `{`, otherwise the path of a JSON file. The value is attached to `Input` as the property `_parameters`. |
| `--output-json-file <path>` | `-o` | Writes `Output` to the file instead of standard output. |
| `--config <path>` | `-c` | The configuration file to read. |
| `--stats` | | Prints the time of each pipeline stage to standard error. |
| `--plugin <paths>` | | Comma-separated paths of plugin assemblies; see [Plugins](/docs/command-line/plugins). |
| `--plugin-directory <dirs>` | `-pd` | Comma-separated directories searched for plugin assemblies, top level only. |
| `--plugin-recursive <dirs>` | `-pr` | Comma-separated directories searched recursively. |

The logging options are listed in the [Overview](/docs/command-line/overview).

## Input

| `-i` | `-p` | `Input` |
| --- | --- | --- |
| no | no | `null` |
| yes | no | The contents of the file, whatever its type |
| no | yes | An object with the single property `_parameters` |
| yes | yes | The contents of the file, which must be an object, with `_parameters` added |

```bash
jyro run -s report.jyro -i orders.json -p '{ "region": "west" }'
```

```jyro
# fragment
var region = Input._parameters.region
Output.orders = WhereByField(Input.orders, "region", "==", region)
```

## Output

`Output` is printed as JSON indented by two spaces, followed by a line end, or written to the `-o` file. When the run fails, the diagnostics are printed to standard error in the form `[JM5200] Ln 2, Col 16: Division by zero`, the partial `Output` is still printed or written, and the exit code is 1.

The script runs with the standard library, the `WriteLog` function described under [Plugins](/docs/command-line/plugins), and the functions of any plugins. The resource limits are the defaults of the host API: 5 seconds, 10,000 statements, 1,000 loop iterations, and a call depth of 64.

## Artifacts

A `.jyrx` file is loaded with the same function set. The artifact is validated and linked when it is loaded, as described in [Artifacts](/docs/host-api/compiling/artifacts). An artifact compiled with a plugin function is run with the same plugin supplied.

## Statistics

With `--stats`, the stage times are printed to standard error after the run: Parse, Validate, Link, Compile, and Execute for a source file, and Deserialize in place of Parse for an artifact.
