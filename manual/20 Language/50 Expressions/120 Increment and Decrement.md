# Increment and Decrement

`++` adds one to its target and `--` subtracts one. Each has a prefix form and a postfix form. The prefix form evaluates to the new value, and the postfix form to the old value. The target must be a variable, a property, or an element, and its value must be a number.

```jyro
var n = 5
Output.pre = ++n
Output.post = n++
Output.n = n
var o = { count: 0 }
o.count++
--o.count
o.count += 1
Output.o = o
```

```json
{ "pre": 6, "post": 6, "n": 7, "o": { "count": 1 } }
```

A target that is not a variable, a property, or an element is refused with JM3204 `InvalidIncrementTarget`.

```jyro
5++
```

```text
[JM3204] Ln 1, Col 1: Invalid increment or decrement target
```

An increment or decrement may be used as a statement or inside an expression. A `++` or `--` at the start of a line is read as a prefix operator on the expression that follows it, never as a postfix operator on the line before it.

```jyro
var n = 1
++n
Output.n = n
```

```json
{ "n": 2 }
```

A string target is refused with JM3301 `StringOperandInAddition` when the script is compiled, or with JM5210 when it runs. A target declared with the type `number` keeps that type.
