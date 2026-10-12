# Reference Semantics

Arrays, objects, and functions are shared by reference. Null, booleans, numbers, and strings are immutable, so the distinction does not apply to them.

## Sharing

Assigning an array or an object to a variable, passing it as an argument, storing it in another container, and returning it from a function all share the same value. A change made through one reference is visible through every other.

```jyro
var a = [1, 2]
var b = a
b[0] = 10
Output.a = a
var o = { inner: { n: 1 } }
var inner = o.inner
inner.n = 2
Output.o = o
```

```json
{ "a": [10, 2], "o": { "inner": { "n": 2 } } }
```

A function parameter receives the same reference as the argument, so a function can change the contents of an array or object it is given.

```jyro
func Mark(item)
    item.seen = true
end
var task = { name: "a" }
Mark(task)
Output.task = task
```

```json
{ "task": { "name": "a", "seen": true } }
```

## Copying

The standard library function `Clone` returns a deep copy. A standard library function that returns a new container, such as `Append` or `Reverse`, copies the container but not the values inside it.

```jyro
var a = [{ n: 1 }]
var b = Clone(a)
b[0].n = 2
var c = Reverse(a)
c[0].n = 3
Output.a = a
Output.b = b
```

```json
{ "a": [{ "n": 3 }], "b": [{ "n": 2 }] }
```

## Iteration snapshots

A `foreach` loop and the standard library functions that take a lambda iterate over the elements the array held when the loop or the call started. A change to the array during iteration does not change which elements are visited.

```jyro
var a = [1, 2]
var seen = []
foreach x in a do
    a = Append(a, x)
    seen = Append(seen, x)
end
Output.seen = seen
Output.a = a
```

```json
{ "seen": [1, 2], "a": [1, 2, 1, 2] }
```

## Read-only values

The value of `Input` is frozen before the script runs. A write to it, or to any container reached from it, is refused. An assignment through `Input` itself is refused at compile time with JM3508 `InputIsReadOnly`; a write through another reference to the same container is refused at run time with JM5007 `InputValueIsReadOnly`.

```jyro
# input: { "list": [1, 2] }
var x = Input.list
x[0] = 9
```

```text
[JM5007] Ln 3, Col 1: Cannot write to Input.
```

`Clone` returns a writable copy of a value taken from `Input`.

## Cycles

A container may contain itself. Such a value can be built and read, but an operation that walks the whole value, such as `==`, `Clone`, or the conversion of `Output` to JSON, exceeds the nesting limit and is refused with JM5910 `ValueNestingLimitExceeded`.

```jyro
var x = {}
x.self = x
Output.a = x == x
```

```text
[JM5910] Ln 3, Col 12: Exceeded the maximum value nesting depth of 128
```
