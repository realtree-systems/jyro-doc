# Equality and Ordering

## Equality

The operators `==` and `!=` compare any two values and never raise an error. Values of different types are never equal.

| Types | Equal when |
| --- | --- |
| null and null | Always |
| boolean and boolean | Both `true` or both `false` |
| number and number | The numbers compare equal; see [Numbers](/docs/language/types-and-values/numbers) |
| string and string | The code units are the same |
| array and array | Same length, and the elements are equal in order |
| object and object | Same keys, and the value of each key is equal; order is ignored |
| function and function | The same function value. Each evaluation of a lambda expression makes a new value |
| Any other pair | Never |

```jyro
Output.a = 0 == false
Output.b = 0 == "0"
Output.c = "" == null
Output.d = [1, 2] == [1, 2.0]
Output.e = { a: [1] } == { a: [1] }
Output.f = (x => x) == (x => x)
```

```json
{ "a": false, "b": false, "c": false, "d": true, "e": true, "f": false }
```

Comparing arrays and objects is deep. A value that contains itself exceeds the nesting limit when compared and is refused with JM5910 `ValueNestingLimitExceeded`.

The `switch` statement and the standard library functions `Contains`, `IndexOf`, and `Distinct` use the same equality.

## Ordering

The operators `<`, `<=`, `>`, and `>=` compare two numbers or two strings. Strings are ordered by their code units. Any other pair of operands is a runtime error.

```jyro
Output.a = 1 < 2
Output.b = "B" < "a"
Output.c = "10" < "9"
```

```json
{ "a": true, "b": true, "c": true }
```

```jyro
Output.a = 1 < "2"
```

```text
[JM5206] Ln 1, Col 12: Unsupported binary operation < for number and string
```

## Sort order

The standard library functions `Sort`, `SortBy`, and `SortByField` use a total order over all values, which never raises an error. Values are ordered first by type, then within a type by value.

| Rank | Type | Order within the type |
| --- | --- | --- |
| 1 | null | |
| 2 | boolean | `false` before `true` |
| 3 | number | By value; an exact number before an inexact number of the same value |
| 4 | string | By code units |
| 5 | array | Arrays are not ordered among themselves |
| 6 | object | Objects are not ordered among themselves |
| 7 | function | Functions are not ordered among themselves |

```jyro
Output.sorted = Sort([{}, "b", 2, null, true, [1], "a", 1, false])
```

```json
{ "sorted": [null, false, true, 1, 2, "a", "b", [1], {}] }
```
