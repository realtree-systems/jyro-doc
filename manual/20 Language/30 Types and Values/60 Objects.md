# Objects

An object is an ordered map from string keys to values. Properties keep the order in which they were added. A property that is deleted and added again is placed last. Objects are mutable and are shared by reference; see [Reference Semantics](/docs/language/types-and-values/reference-semantics).

## Reading a property

A property is read with `.` followed by a name, or with a string in square brackets. The two forms are equivalent, and the bracket form accepts any string.

| Expression | Result |
| --- | --- |
| `o.name` or `o["name"]`, where `o` has the property | The value |
| `o.name` or `o["name"]`, where `o` is an object without the property | `null` |
| `o.name`, where `o` is not an object | `null` |
| `o[k]`, where `o` is an object and `k` is not a string | `null` |

A missing property gives `null` and never a runtime error, so a path such as `Input.customer.address.city` evaluates to `null` when any part is missing.

```jyro
var o = { name: "Ada", "year-born": 1815 }
Output.a = o.name
Output.b = o["year-born"]
Output.c = o.missing
Output.d = o.missing.deeper
Output.e = (5).x
Output.f = o[0]
```

```json
{ "a": "Ada", "b": 1815, "c": null, "d": null, "e": null, "f": null }
```

## Writing a property

An assignment to `o.name` or `o["name"]` adds the property or replaces its value. The target must be an object.

```jyro
var o = { a: 1 }
o.a = 10
o.b = 2
o["c d"] = 3
Output.o = o
```

```json
{ "o": { "a": 10, "b": 2, "c d": 3 } }
```

```jyro
Output.x.y = 1
```

```text
[JM5306] Ln 1, Col 1: Cannot set property 'y' on value of type null
```

An index that is not a string is refused with JM5308 `InvalidIndexType`.

```jyro
var o = { k: 1 }
o[0] = 1
```

```text
[JM5308] Ln 2, Col 1: Cannot write to a value of type object at an index of type number; an array is indexed by a number and an object by a string
```

## Deleting a property

The `delete` statement removes a property. Deleting a property that does not exist has no effect. See [Delete](/docs/language/statements/delete).

```jyro
var o = { k: 1, j: 2 }
delete o.k
delete o.missing
o.k = 3
Output.o = o
```

```json
{ "o": { "j": 2, "k": 3 } }
```

## Equality

Two objects are equal when they have the same keys with equal values. The order of the properties does not matter.

```jyro
Output.a = { a: 1, b: 2 } == { b: 2, a: 1 }
Output.b = { a: 1 } == { a: 1, b: null }
```

```json
{ "a": true, "b": false }
```

## Keys and values

The standard library functions `Keys`, `Values`, and `HasProperty` enumerate and test properties. A `foreach` loop over an object visits its values in property order; see [Foreach](/docs/language/statements/foreach).
