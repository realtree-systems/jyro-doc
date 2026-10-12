<!-- hand-written -->
# Regular Expressions

`RegexTest`, `RegexMatch`, `RegexMatchAll`, and `RegexMatchDetail` take a pattern as a string. `ValidateSchema` reads the `pattern` and `patternProperties` keywords of a schema the same way. Patterns are matched by the language's own engine, which reads them as ECMA-262 (ECMAScript 2024) regular expressions with no flags and without the Annex B extensions. Text and patterns are read as UTF-16 code units.

```jyro
Output.a = RegexTest("order-123", "^[a-z]+-[0-9]+$")
Output.b = RegexMatch("2024-01-15", "[0-9]{4}")
Output.c = RegexMatchAll("a1b22c333", "[0-9]+")
Output.d = RegexMatchDetail("John Smith", "(?<first>\\w+) (?<last>\\w+)")
```

```json
{
  "a": true,
  "b": "2024",
  "c": ["1", "22", "333"],
  "d": { "match": "John Smith", "index": 0, "groups": ["John", "Smith"] }
}
```

A pattern is written inside a string literal, so a backslash in the pattern is written as `\\`.

## Supported syntax

| Construct | Syntax |
| --- | --- |
| Any character except a line terminator | `.` |
| Start and end of the text | `^`, `$` |
| Word boundary | `\b`, `\B` |
| Character class | `[abc]`, `[^abc]`, `[a-z]` |
| Class escapes | `\d`, `\D`, `\w`, `\W`, `\s`, `\S` |
| Character escapes | `\f`, `\n`, `\r`, `\t`, `\v`, `\0`, `\cX`, `\xHH`, `\uHHHH`, and an escaped punctuation character |
| Groups | `(…)`, `(?:…)`, `(?<name>…)` |
| Back references | `\1` to `\9` and beyond, `\k<name>` |
| Lookahead | `(?=…)`, `(?!…)` |
| Lookbehind | `(?<=…)`, `(?<!…)` |
| Quantifiers | `*`, `+`, `?`, `{n}`, `{n,}`, `{n,m}`, each with a lazy form ending in `?` |
| Alternation | `a\|b` |

`^` and `$` match only at the start and the end of the whole text. `\s` matches the white space characters of ECMAScript, and `\w` the ASCII word characters.

## Unsupported syntax

The following are not accepted and make the pattern invalid:

- Flags of any kind. There is no case-insensitive, multi-line, dot-all, sticky, or Unicode mode.
- `\p{…}` property escapes and `\u{…}` code point escapes.
- A quantifier applied to a lookahead or a lookbehind.
- A lone `{`, `}`, or `]`, and a class range with a class escape at either end.
- A back reference to a group that does not exist.

## Errors and limits

| Condition | Diagnostic |
| --- | --- |
| The pattern is not valid | JM5701 `RegexInvalidPattern` |
| One evaluation runs longer than the regular expression time limit | JM5700 `RegexTimeout` |

```jyro
Output.a = RegexTest("abc", "(")
```

```text
[JM5701] Ln 1, Col 12: RegexTest(): Invalid regex pattern '('
```

The time limit for one evaluation is one second unless the host sets another with `WithMaxRegexTime`; see [Resource Limits](/docs/host-api/running/resource-limits). Compiling and matching a pattern also check the run's execution time limit and host cancellation.
