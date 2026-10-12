# Foreach

```jyro
# fragment
foreach name in expression do
    statements
end
```

A `foreach` loop runs the block once for each element of a collection.

| Collection | Values visited |
| --- | --- |
| array | The elements, in order |
| object | The property values, in property order |
| string | Each UTF-16 code unit, as a string of length one |

Any other value is a runtime error with JM5402 `NotIterable`. A literal `null`, number, or boolean in the collection position is refused when the script is compiled, with JM3402 `NotIterableLiteral`.

```jyro
# input: { "names": ["a", "b"], "ages": { "a": 30, "b": 40 } }
var upper = []
foreach name in Input.names do
    upper = Append(upper, ToUpper(name))
end
var total = 0
foreach age in Input.ages do
    total += age
end
var units = []
foreach u in "a😀" do
    units = Append(units, Length(u))
end
Output.upper = upper
Output.total = total
Output.units = units
```

```json
{ "upper": ["A", "B"], "total": 70, "units": [1, 1, 1] }
```

```jyro
# input: { "n": 5 }
foreach x in Input.n do
end
```

```text
[JM5402] Ln 2, Col 1: Value of type number is not iterable
```

An object yields its values only. The keys are obtained with the standard library function `Keys`.

```jyro
# input: { "ages": { "a": 30, "b": 40 } }
var lines = []
foreach key in Keys(Input.ages) do
    lines = Append(lines, key & "=" & Input.ages[key])
end
Output.lines = lines
```

```json
{ "lines": ["a=30", "b=40"] }
```

## Snapshot

The collection is evaluated once, and the loop visits the elements it held at that moment. A change to the collection inside the loop does not change which elements are visited.

## The loop variable

The loop variable is visible only in the body and is a fresh variable in each iteration. It holds a reference to the element, so a change to an object or array element through it changes the collection.

```jyro
var items = [{ n: 1 }, { n: 2 }]
foreach item in items do
    item.n *= 10
end
Output.items = items
```

```json
{ "items": [{ "n": 10 }, { "n": 20 }] }
```

`break` ends the loop and `continue` advances to the next element; see [Break and Continue](/docs/language/statements/break-and-continue). Each iteration counts toward the run's loop iteration limit.
