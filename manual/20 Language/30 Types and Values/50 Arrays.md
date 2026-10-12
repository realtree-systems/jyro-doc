# Arrays

An array is an ordered list of values. Elements may be of any type and need not share a type. Arrays are mutable and are shared by reference; see [Reference Semantics](/docs/language/types-and-values/reference-semantics).

## Reading an element

An element is read with an integer index in square brackets. Indexes start at zero.

| Expression | Result |
| --- | --- |
| `a[i]`, where `i` is an integer from 0 to `Length(a) - 1` | The element |
| `a[i]`, where `i` is negative or at least `Length(a)` | `null` |
| `a[i]`, where `i` is a number with a fraction | JM5310 `IntegerIndexRequired` |
| `a[i]`, where `i` is not a number | `null` |

```jyro
var a = [10, 20, 30]
Output.first = a[0]
Output.last = a[2]
Output.past = a[3]
Output.negative = a[-1]
Output.text = a["1"]
```

```json
{ "first": 10, "last": 30, "past": null, "negative": null, "text": null }
```

```jyro
var a = [10, 20, 30]
Output.x = a[1.5]
```

```text
[JM5310] Ln 2, Col 12: An index must be an integer. Received: 1.5
```

## Writing an element

An assignment to `a[i]` replaces the element at `i`, or appends an element when `i` equals `Length(a)`. Any other index is a runtime error; an array is never extended with gaps.

```jyro
var a = [1, 2]
a[0] = 10
a[2] = 3
Output.a = a
```

```json
{ "a": [10, 2, 3] }
```

```jyro
var a = [1]
a[3] = 1
```

```text
[JM5300] Ln 2, Col 1: array position 3 is out of bounds for a length of 1
```

An index that is not a number is refused with JM5308 `InvalidIndexType`.

```jyro
var a = [1]
a["x"] = 1
```

```text
[JM5308] Ln 2, Col 1: Cannot write to a value of type array at an index of type string; an array is indexed by a number and an object by a string
```

## Removing elements

The `delete` statement does not apply to arrays. Elements are removed with the standard library functions `RemoveAt`, `RemoveFirst`, and `RemoveLast`, which return a new array.

```jyro
var a = [1]
delete a[0]
```

```text
[JM5309] Ln 2, Col 1: Cannot delete '0' from a value of type array; delete removes an object property (use RemoveAt for an array)
```

## Equality

Two arrays are equal when they have the same length and their elements are equal in order.

```jyro
Output.a = [1, 2] == [1, 2]
Output.b = [1, 2] == [2, 1]
Output.c = [1, [2]] == [1, [2]]
```

```json
{ "a": true, "b": false, "c": true }
```

## Iteration

A `foreach` loop visits the elements of an array in order, from a snapshot of the array taken when the loop starts. See [Foreach](/docs/language/statements/foreach).
