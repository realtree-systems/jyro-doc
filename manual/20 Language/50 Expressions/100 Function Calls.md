# Function Calls

A call is a function name followed by an argument list in parentheses. The name must be the name of a function declared in the script with `func`, a function in a library attached by the host, a standard library function, a host function, or a variant constructor.

```jyro
func Double(n)
    return n * 2
end
Output.a = Double(4)
Output.b = ToUpper("abc")
```

```json
{ "a": 8, "b": "ABC" }
```

The callee must be a bare name. A call on any other expression, including a variable that holds a lambda, is refused. A name that is not a function is refused with JM4100 `UndefinedFunction`.

```jyro
Output.a = (x => x + 1)(1)
```

```text
[JM2003] Ln 1, Col 24: A call must be made on a function name, not on another expression
```

```jyro
var f = x => x
Output.a = f(1)
```

```text
[JM4100] Ln 2, Col 12: Undefined function 'f'
```

A function name is not a value: it cannot be assigned to a variable or passed as an argument. A lambda that calls the function is used instead; see [Lambdas](/docs/language/expressions/lambdas).

## Positional arguments

Arguments are matched to parameters in order. The number of arguments must be between the number of required parameters and the total number of parameters; the check is made when the script is compiled.

```jyro
Output.a = Substring("hello", 1)
Output.b = Substring("hello", 1, 3)
```

```json
{ "a": "ello", "b": "ell" }
```

```jyro
Output.a = Length()
```

```text
[JM4200] Ln 1, Col 12: Function 'Length' requires at least 1 arguments, but 0 were provided
```

```jyro
Output.a = Length([1], 2)
```

```text
[JM4201] Ln 1, Col 12: Function 'Length' accepts at most 1 arguments, but 2 were provided
```

## Named arguments

Each argument may be written as `name: value`, where `name` is a parameter name. Named arguments may be given in any order. A call uses either positional arguments or named arguments; mixing the two is a parse error.

```jyro
Output.a = Substring(start: 1, text: "hello")
Output.b = PadLeft(padding: "0", length: 5, text: "42")
```

```json
{ "a": "ello", "b": "00042" }
```

```jyro
Output.a = Substring(text: "a", text: "b", start: 0)
```

```text
[JM4202] Ln 1, Col 12: Duplicate named argument 'text' in call to 'Substring'
```

```jyro
Output.a = Substring(text: "a", begin: 0)
```

```text
[JM4203] Ln 1, Col 12: Unknown parameter 'begin' in call to 'Substring'
[JM4204] Ln 1, Col 12: Required parameter 'start' not provided in call to 'Substring'
```

## Optional parameters

An optional parameter that is not given receives `null`. Passing `null` explicitly has the same effect. A standard library function treats such a parameter as absent and applies its default.

```jyro
Output.a = Substring("hello", 1, null)
Output.b = Slice(items: [1, 2, 3], start: 1, finish: null)
```

```json
{ "a": "ello", "b": [2, 3] }
```

## Evaluation

Arguments are evaluated from left to right before the function is entered. A call counts one level of call depth; see [Resource Limits](/docs/language/errors-and-limits/resource-limits).

A `(` at the start of a line starts a new statement. The argument list must open on the same line as the function name.
