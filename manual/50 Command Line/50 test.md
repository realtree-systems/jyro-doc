# test

Runs a script and compares its `Output` with expected JSON.

```bash
jyro test -s <script> -o <expected.json> [-i <input.json>] [-p <parameters>] [options]
```

| Option | Alias | Effect |
| --- | --- | --- |
| `--input-script-file <path>` | `-s` | The script or artifact to run. Required. |
| `--output-json-file <path>` | `-o` | The JSON file holding the expected `Output`. Required. |
| `--input-json-file <path>` | `-i` | A JSON file whose contents become `Input`. |
| `--parameters <json\|path>` | `-p` | Inline JSON or a file path, attached to `Input` as `_parameters`; see [run](/docs/command-line/run). |
| `--stats` | | Prints the stage times to standard error. |
| `--plugin <paths>`, `--plugin-directory <dirs>`, `--plugin-recursive <dirs>` | , `-pd`, `-pr` | Plugin assemblies; see [Plugins](/docs/command-line/plugins). |

The logging options are listed in the [Overview](/docs/command-line/overview). Log output goes to standard error. The command does not read the configuration file and does not take `--stdin`.

## Comparison

The actual and expected values are compared structurally.

| Values | Equal when |
| --- | --- |
| Objects | They have the same keys, in any order, with equal values |
| Arrays | They have the same length and equal elements in order |
| Numbers | They are within a relative tolerance of 1e-10, or within 1e-15 when either is zero |
| Strings, booleans, null | They are the same |

Each difference is reported with a path such as `$.orders[2].total`; a length difference between arrays is reported at `$.orders.length`. The exit code is 0 when there is no difference and 1 otherwise. A script that fails to compile or run is reported with its diagnostics and exits with 1.

```bash
jyro test -s report.jyro -i orders.json -o expected.json
```
