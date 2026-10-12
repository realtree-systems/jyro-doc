# Source Text

A script is Unicode text. The host supplies it as a string; the command line tool reads it from a UTF-8 file.

## Byte order mark

A U+FEFF at the first position of the source is ignored. A U+FEFF anywhere else is a parse error.

## White space

Any character with the Unicode White_Space property separates tokens. White space has no other meaning except where a line end is significant, as described below.

## Line ends

A line ends at LF or at CRLF. A CR on its own is white space.

## Statements and lines

Statements are not separated by punctuation. There is no statement terminator, and a statement need not start on a new line.

```jyro
var a = 1 var b = 2 Output.sum = a + b
```

```json
{ "sum": 3 }
```

An expression may continue across a line end. A line that begins with an operator, a `.`, or a `?` continues the expression on the line before it.

```jyro
var total = 10
    - 3
Output.total = total
Output.name = { first: "Ada" }
    .first
```

```json
{ "total": 7, "name": "Ada" }
```

A line that begins with `(`, `[`, `++`, or `--` starts a new statement. It does not continue the expression on the line before it as a call, an index, or a postfix operator.

```jyro
var items = [1, 2, 3]
var x = items
[4]
Output.x = x
var n = 1
++n
Output.n = n
```

```json
{ "x": [1, 2, 3], "n": 2 }
```

Inside an argument list, an array literal, an object literal, or parentheses, a line that begins with `(`, `[`, `++`, or `--` is a parse error.

## Values after return, exit, and fail

The keywords `return`, `exit`, and `fail` take an optional value. The value is taken only when it starts on the same line as the keyword. A line end, a comment, the end of the source, or one of the keywords `end`, `elseif`, `else`, `case`, and `default` directly after the keyword means there is no value.

```jyro
func Nothing()
    return
    42
end
Output.first = Nothing()
```

```json
{ "first": null }
```

A value that starts on the keyword's line may continue onto the following lines.

```jyro
exit (
    "done")
```

```text
Exit message: done
```

## Nesting

Expressions, statements, and literals may be nested to a depth of 256. Deeper source is refused with JM2002 `SourceNestingLimitExceeded`.
