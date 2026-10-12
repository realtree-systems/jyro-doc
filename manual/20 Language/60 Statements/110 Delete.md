# Delete

```jyro
# fragment
delete object.name
delete object[key]
```

`delete` removes a property from an object. The target must be a property access or an index access on an object; any other target is refused when the script is compiled, with JM3201 `InvalidDeleteTarget`. Deleting a property that does not exist has no effect.

```jyro
var o = { a: 1, b: 2, c: 3 }
delete o.a
delete o["c"]
delete o.missing
Output.o = o
```

```json
{ "o": { "b": 2 } }
```

```jyro
var o = { k: 1 }
delete o
```

```text
[JM3201] Ln 2, Col 1: Invalid delete target
```

`delete` does not apply to arrays. An element is removed with the standard library functions `RemoveAt`, `RemoveFirst`, or `RemoveLast`.

```jyro
var a = [1, 2]
delete a[0]
```

```text
[JM5309] Ln 2, Col 1: Cannot delete '0' from a value of type array; delete removes an object property (use RemoveAt for an array)
```

A key that is not a string is refused with JM5308 `InvalidIndexType`. A target that starts with `Input` is refused when the script is compiled, with JM3508 `InputIsReadOnly`.
