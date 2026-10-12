# Comments

A comment starts with `#` and runs to the end of the line. A comment may follow a statement on the same line.

```jyro
# The whole line is a comment.
Output.value = 1   # The rest of the line is a comment.
```

```json
{ "value": 1 }
```

There is no block comment. A `#` inside a string literal is part of the string.

```jyro
Output.tag = "#1"
```

```json
{ "tag": "#1" }
```
