# validate

Verifies that a `.jyrx` artifact was compiled from a given `.jyro` source file.

```bash
jyro validate --compiled <script.jyrx> --raw <script.jyro> [options]
```

| Option | Effect |
| --- | --- |
| `--compiled <path>` | The artifact. Required. |
| `--raw <path>` | The source file. Required. |

The logging options are listed in the [Overview](/docs/command-line/overview).

The command reads the SHA-256 recorded in the artifact's header and computes the SHA-256 of the source file's text as UTF-8. When the two are equal it prints `[PASS]` and exits with 0. Otherwise it prints `[FAIL]` with both hashes in hexadecimal and exits with 1.

```text
[PASS]: report.jyrx is a valid representation of report.jyro
```

The check compares the source text only. A change to the source, including a change to white space or a comment, gives a different hash. The check does not run or load the artifact; an artifact that cannot be read is reported as an error.
