# JyroArray and JyroObject

## JyroArray

`JyroArray` is an ordered, mutable list of values. It implements `IReadOnlyList<JyroValue>` and has a collection initializer.

| Member | Description |
| --- | --- |
| `Count` | The number of elements |
| `this[int]` | Reads or replaces an element; an index out of range throws |
| `TryGetItem(int, out JyroValue)` | Reads an element when the index is in range |
| `Add(JyroValue)` | Appends |
| `Insert(int, JyroValue)` | Inserts at an index from 0 to `Count` |
| `RemoveAt(int)` | Removes an element |
| `Clear()` | Removes every element |
| `Contains(JyroValue)`, `IndexOf(JyroValue)` | Search by `EqualsValue` |

```csharp
using RealTree.Jyro;

var scores = new JyroArray { new JyroNumber(72), new JyroNumber(88) };
scores.Add(new JyroNumber(95));
scores.Insert(0, new JyroNumber(60));
scores.RemoveAt(1);

Console.WriteLine(scores.ToJson()); // [60,88,95]
Console.WriteLine(scores.Count);    // 3
```

## JyroObject

`JyroObject` is an ordered, mutable map from string keys to values. Properties keep insertion order; a removed key that is added again is placed last. It implements `IReadOnlyDictionary<string, JyroValue>` and has a collection initializer.

| Member | Description |
| --- | --- |
| `Count`, `Keys`, `Values` | The properties, in order |
| `this[string]` | Reads a property, or throws `KeyNotFoundException` when absent; writes add or replace |
| `GetProperty(string)` | The value, or `JyroNull` when absent |
| `TryGetValue(string, out JyroValue)`, `ContainsKey(string)` | Lookup |
| `Add(string, JyroValue)` | Adds a property; throws `ArgumentException` when the key exists |
| `SetProperty(string, JyroValue)` | Adds or replaces |
| `Remove(string)` | Removes a property; returns whether it existed |
| `Clear()` | Removes every property |
| `ToDictionary()` | A `Dictionary<string, object>` of the properties converted with `ToObjectValue` |

```csharp
using RealTree.Jyro;

var order = new JyroObject();
order.Add("id", new JyroNumber(42));
order.SetProperty("status", new JyroString("open"));
order["status"] = new JyroString("shipped");
order.Remove("id");
order.Add("id", new JyroNumber(42));

Console.WriteLine(order.ToJson());                   // {"status":"shipped","id":42}
Console.WriteLine(order.GetProperty("missing").IsNull); // True
```

## Reading a result

A run's value is read through these members after a downcast with `AsObject()` or `AsArray()`.

```csharp
using RealTree.Jyro;

CompiledProgram program = new JyroCompiler()
    .WithStdlib()
    .WithSource("Output.names = Map(Input, p => p.name) Output.count = Length(Input)")
    .CompileOrThrow();

JyroValue output = new JyroRunner(program)
    .WithInputJson("""[ { "name": "Ada" }, { "name": "Alan" } ]""")
    .ExecuteOrThrow();

JyroObject result = output.AsObject();
long count = result["count"].ToInt64();
foreach (JyroValue name in result["names"].AsArray())
    Console.WriteLine(name.ToStringValue());
```

## Frozen values

`Freeze()` makes a value and every container reached from it read-only; `IsFrozen` reports the state. The input of a run is frozen when it is set. A write to a frozen array or object throws `JyroRuntimeException` with JM5005 `ValueIsReadOnly`, or JM5007 `InputValueIsReadOnly` when the value is a run's `Input`. Freezing cannot be undone.
