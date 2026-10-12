# Return

```jyro
# fragment
return
return expression
```

`return` ends the current function call. With an expression, the value of the expression is the value of the call. Without one, the value is `null`. A function whose body ends without a `return` also gives `null`.

```jyro
func Sign(n)
    if n > 0 then
        return "positive"
    end
    if n < 0 then
        return "negative"
    end
    return "zero"
end
func Nothing()
    return
end
func NoReturn()
    var unused = 1
end
Output.a = Sign(-3)
Output.b = Nothing()
Output.c = NoReturn()
```

```json
{ "a": "negative", "b": null, "c": null }
```

The expression must start on the same line as `return`. A `return` followed by a line end returns `null`, and the next line is the next statement; see [Source Text](/docs/language/lexical-structure/source-text).

`return` is valid only inside a function body. At the top level of a script, `exit` ends the run; see [Exit](/docs/language/statements/exit).

```jyro
return 1
```

```text
[JM3505] Ln 1, Col 1: 'return' is only valid inside a function
```
