# Input and Output

`Input` and `Output` are the only predefined names. They are supplied by the host and cannot be declared, used as a parameter name, or used as the name of a function, a union, or a variant.

```jyro
var Input = 1
```

```text
[JM3103] Ln 1, Col 1: Cannot bind 'Input'; Input and Output are supplied by the host
```

The lowercase names `input` and `output` are ordinary identifiers.

## Input

`Input` holds the value the host supplies for the run. It may be of any type, and it is `null` when the host supplies nothing. `Input` is read-only: the value is frozen before the script runs, and every container reached from it is frozen with it.

An assignment, a `delete`, or an increment whose target starts with `Input` is refused when the script is compiled.

```jyro
Input.count = 1
```

```text
[JM3508] Ln 1, Col 1: Cannot modify Input.
```

A write through another reference to a container reached from `Input` is refused when the script runs.

```jyro
# input: { "list": [1, 2] }
var list = Input.list
list[0] = 9
```

```text
[JM5007] Ln 3, Col 1: Cannot write to Input.
```

The standard library function `Clone` returns a writable copy.

```jyro
# input: { "list": [1, 2] }
var list = Clone(Input.list)
list[0] = 9
Output.list = list
```

```json
{ "list": [9, 2] }
```

`Input` may be read inside a function or a lambda.

## Output

`Output` starts as an empty object, unless the host supplies a starting value. Its value when the script ends, or when `exit` or `fail` runs, is the result of the run. `Output` may be replaced with a value of any type.

```jyro
Output = [1, 2, 3]
```

```json
[1, 2, 3]
```

`Output` cannot be referenced inside a function or a lambda body, for reading or for writing.

```jyro
func Record(value)
    Output.value = value
end
```

```text
[JM3504] Ln 2, Col 5: Cannot access Output inside a function or lambda body
```

A function returns its result with `return`. A function can change the contents of `Output` when `Output` is passed to it as an argument, since the argument is a reference to the same object.

```jyro
func Record(target, value)
    target.value = value
end
Record(Output, 42)
```

```json
{ "value": 42 }
```
