# Declaring Functions

```jyro
# fragment
func Name(parameter, parameter: type, parameter = literal, parameter: type = literal)
    statements
end
```

A function is declared with `func`, a name, a parameter list in parentheses, a body, and `end`. The parameter list may be empty. A function is called by its name; see [Function Calls](/docs/language/expressions/function-calls).

```jyro
func Area(width: number, height: number = 1)
    return width * height
end
Output.a = Area(3, 4)
Output.b = Area(5)
```

```json
{ "a": 12, "b": 5 }
```

## Placement and visibility

A function must be declared at the top level of the script, outside every block and every other function. A function is visible throughout the script, including before its declaration, so functions may call each other in any order and may be recursive.

```jyro
func IsEven(n)
    if n == 0 then
        return true
    end
    return IsOdd(n - 1)
end
func IsOdd(n)
    if n == 0 then
        return false
    end
    return IsEven(n - 1)
end
Output.a = IsEven(10)
Output.b = Later()
func Later()
    return "declared after the call"
end
```

```json
{ "a": true, "b": "declared after the call" }
```

```jyro
if true then
    func F()
        return 1
    end
end
```

```text
[JM3500] Ln 2, Col 5: Function 'F' must be declared at the top level
```

```jyro
func Outer()
    func Inner()
        return 1
    end
    return 1
end
```

```text
[JM3501] Ln 2, Col 5: Function declarations cannot be nested inside other functions
```

## Parameters

A parameter may have a type. The argument is converted to that type on entry, as described in [Type Coercion](/docs/language/types-and-values/type-coercion).

A parameter may have a default value, which makes it optional. The default must be a literal, or `-` applied to a number literal. An optional parameter receives its default when the argument is omitted, and also when the argument is `null`. Required parameters must come before optional parameters.

```jyro
func Label(n: number = -1, text: string = "none")
    return n & ":" & text
end
Output.a = Label()
Output.b = Label(null, null)
Output.c = Label("5")
Output.d = Label(text: "set")
```

```json
{ "a": "-1:none", "b": "-1:none", "c": "5:none", "d": "-1:set" }
```

```jyro
func F(a = 1, b)
    return 1
end
```

```text
[JM3506] Ln 1, Col 1: Required parameter 'b' cannot follow optional parameter 'a' in function 'F'
```

```jyro
func F(a = 1 + 1)
    return 1
end
```

```text
[JM3507] Ln 1, Col 1: Default value for parameter 'a' in function 'F' must be a literal
```

Parameter names must be distinct, and a parameter cannot be named `Input` or `Output` or with a reserved keyword.

```jyro
func F(a, a)
    return 1
end
```

```text
[JM3502] Ln 1, Col 1: Duplicate parameter 'a' F
```

## Names

A function name must be unique among the functions the script can call.

| Conflict | Diagnostic |
| --- | --- |
| The same name declared twice in the script | JM4101 `DuplicateFunction` |
| The name of a standard library function or a host function | JM4102 `FunctionOverride` |
| The name of a function in an attached library | JM4103 `FunctionShadowsLibrary` |
| The name of a union variant | JM4301 `FunctionShadowsVariant` |

The standard library names are reserved whether or not the host has enabled the standard library.

```jyro
func Length(x)
    return 1
end
```

```text
[JM4102] Ln 1, Col 1: Function 'Length' overrides a built-in function
```

```jyro
func F()
    return 1
end
func F()
    return 2
end
```

```text
[JM4101] Ln 4, Col 1: Function 'F' is already defined
```
