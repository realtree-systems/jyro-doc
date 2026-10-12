<!-- hand-written -->
# Standard Library Overview

The standard library is a set of functions available to every script when the host enables it. The functions are grouped by category.

| Category | Functions |
| --- | --- |
| [String](/docs/standard-library/string/overview) | Case, trimming, searching, splitting, joining, padding, regular expressions, and random text |
| [Array](/docs/standard-library/array/overview) | Length, adding and removing elements, slicing, sorting, grouping, and ranges |
| [Math](/docs/standard-library/math/overview) | Rounding, aggregates, powers, roots, logarithms, and random integers |
| [Date and Time](/docs/standard-library/date-and-time/overview) | The current time, parsing, formatting, arithmetic, and parts of a date |
| [Utility](/docs/standard-library/utility/overview) | Types, conversions, object keys and values, merging, cloning, JSON, Base64, and identifiers |
| [Schema](/docs/standard-library/schema/overview) | Required-field checks and JSON Schema validation |
| [Query](/docs/standard-library/query/overview) | Filtering, finding, counting, and projecting arrays of objects by a field |
| [Lambda](/docs/standard-library/lambda/overview) | Mapping, filtering, reducing, and sorting arrays with a lambda |

## Page conventions

Each function has a page with its signature, a table of its parameters, what it returns, and an example. In a signature, a parameter in square brackets is optional.

```jyro
# fragment
Substring(text, start, [length])
```

The type of a parameter is the type the function reads the argument as. The parameter description states what the function does with a value of another type, and with `null`.

## Calling a function

A function is called by name with positional arguments or with named arguments, as described in [Function Calls](/docs/language/expressions/function-calls). The parameter names in the signatures are the names used for named arguments.

```jyro
Output.a = Substring("hello", 1, 3)
Output.b = Substring(text: "hello", start: 1, length: 3)
```

```json
{ "a": "ell", "b": "ell" }
```

An optional parameter that is omitted, or that is given `null`, takes its default. The argument count and the names of named arguments are checked when the script is compiled: JM4200 `TooFewArguments`, JM4201 `TooManyArguments`, JM4202 `DuplicateNamedArgument`, JM4203 `UnknownNamedArgument`, and JM4204 `MissingRequiredNamedArgument`.

## Argument types

Argument types are checked when the function runs. The rules common to the library are:

| Parameter type | Accepted values |
| --- | --- |
| string | A string. A number or a boolean is read as its string form, and `null` as the empty string, unless the parameter description says otherwise. An array, an object, or a function is a runtime error. |
| number | A number. `null` and any other type are a runtime error. Where an integer is required, a number with a fraction is refused with JM5904 `IntegerRequired`, and a negative number where a non-negative one is required with JM5907 `NonNegativeIntegerRequired`. |
| boolean | A boolean. `null` and any other type are a runtime error. |
| array | An array. `null` is read as the empty array. Any other type is a runtime error. |
| object | An object. `null` is read as the empty object. Any other type is a runtime error. |
| function | A lambda with the number of parameters the function calls it with. Any other value, `null` included, is a runtime error; see [Lambdas](/docs/language/expressions/lambdas). |
| any | Any value. |

A value of the wrong type is refused with JM5104 `ArgumentTypeMismatch`.

```jyro
Output.a = ToUpper(42)
Output.b = Length(null)
Output.c = Join([1, true, null], ",")
```

```json
{ "a": "42", "b": 0, "c": "1,true,null" }
```

```jyro
Output.a = Absolute("x")
```

```text
[JM5104] Ln 1, Col 12: Expected number but got string
```

## Results

A function returns a new value and leaves its arguments unchanged. A function that returns an array or an object returns a new container; the values inside it are shared with the arguments, as described in [Reference Semantics](/docs/language/types-and-values/reference-semantics).

A string or an array a function builds is subject to the value size limit of 16,777,216 code units or elements, JM5919 `ValueSizeLimitExceeded`. `PadLeft`, `PadRight`, `RandomString`, and `Range` refuse to build a result longer than 1,000,000 with JM5702 `GeneratedLengthExceeded`.

A call to a standard library function counts as one statement. A long-running call checks the execution time limit and host cancellation as it works, but does not count toward the statement or loop iteration limits.

## Effects

Every function declares an effect. `Now`, `Today`, `NewGuid`, `RandomInt`, `RandomChoice`, and `RandomString` are non-deterministic; `Sleep` is blocking; every other function is pure. A page states the effect when it is not pure. The host may set a ceiling on the effects a script or a library may call; see [Function Effects](/docs/language/functions/function-effects).

## Names

The names of the standard library functions are reserved. A script cannot declare a function or a variant with one of these names, whether or not the host has enabled the library.
