# JyroValue

`JyroValue` is the abstract base of every value a script reads or writes. Each of the seven types has a sealed subclass.

| Class | Type | Construction |
| --- | --- | --- |
| `JyroNull` | null | `JyroNull.Instance` |
| `JyroBoolean` | boolean | `JyroBoolean.True`, `JyroBoolean.False`, `JyroBoolean.FromBoolean(bool)` |
| `JyroNumber` | number | `new JyroNumber(int)`, `(long)`, `(decimal)`, `(double)` |
| `JyroString` | string | `new JyroString(string)` |
| `JyroArray` | array | `new JyroArray()`, then `Add` |
| `JyroObject` | object | `new JyroObject()`, then `Add` or `SetProperty` |
| `JyroFunction` | function | `new JyroFunction(Func<IReadOnlyList<JyroValue>, JyroValue>, int)` |

```csharp
using RealTree.Jyro;

var person = new JyroObject
{
    { "name", new JyroString("Ada") },
    { "age", new JyroNumber(36) },
    { "active", JyroBoolean.True },
    { "tags", new JyroArray { new JyroString("math"), new JyroString("engines") } },
    { "manager", JyroNull.Instance },
};

Console.WriteLine(person.ToJson());
// {"name":"Ada","age":36,"active":true,"tags":["math","engines"],"manager":null}
```

## Members

| Member | Description |
| --- | --- |
| `ValueType` | The `JyroValueType` of the value: `Null`, `Boolean`, `Number`, `String`, `Array`, `Object`, or `Function` |
| `IsNull` | `true` for `JyroNull` |
| `IsTruthy()` | The truthiness the language applies to conditions; never throws |
| `EqualsValue(JyroValue)` | Equality as the `==` operator defines it |
| `AsBoolean()`, `AsNumber()`, `AsString()`, `AsArray()`, `AsObject()`, `AsFunction()` | The value as its subclass, or JM5102 `InvalidCast` |
| `ToStringValue()`, `ToDouble()`, `ToInt32()`, `ToInt64()` | The scalar converted strictly, or `InvalidCast` for a value of another type |
| `ToObjectValue()` | The value as a .NET object; see the table below |
| `ToJson()`, `WriteJson(Utf8JsonWriter)` | The value as JSON |
| `GetProperty(string)`, `SetProperty(string, JyroValue)` | Property access on an object; `GetProperty` gives `JyroNull` for any other value, and `SetProperty` throws for one |
| `Freeze()`, `IsFrozen` | Makes the value and every container reached from it read-only; a write to a frozen value is JM5005 `ValueIsReadOnly` |
| `static CompareForSort(JyroValue, JyroValue)` | The total order the sort functions use |

`ValueType` is an enum and works in a `switch`. `ParameterType`, used in function signatures, has the same members and values with `Any` added, so the two cast to each other.

## FromJson

`JyroValue.FromJson(string)` and `FromJson(Stream)` parse strict RFC 8259 JSON; `FromJsonAsync(Stream, CancellationToken)` is the asynchronous form. A number is exact when its text fits the exact representation and otherwise the nearest double. Text that is not JSON throws `JyroRuntimeException` with JM5917 `InvalidJson`, and nesting deeper than 128 throws JM5910 `ValueNestingLimitExceeded`. `JyroRunner.WithInputJson` reads the same way but reports a fault through `Execute`.

## FromObject

`JyroValue.FromObject(object)` converts a .NET value by its shape. `JyroRunner.WithInput(object)` uses the same rules.

| .NET value | Jyro value |
| --- | --- |
| `null` | null |
| `bool`, `string` | boolean, string |
| Any integer or floating-point number type, including `BigInteger`, `Int128`, and `Half` | number: `decimal` and integers within its range are exact; `double`, `float`, and `Half` are inexact; a larger integer is its nearest double |
| `JsonElement`, `JsonNode` | The JSON it holds |
| Newtonsoft `JToken` | Its JSON; a token with no JSON form becomes its text |
| `IDictionary<string, T>`, `IReadOnlyDictionary<string, T>` | object, in enumeration order |
| A dictionary with another key type | array of objects with `Key` and `Value` |
| Any other `IEnumerable` | array |
| `DateTime`, `DateTimeOffset`, `DateOnly`, `TimeOnly`, `TimeSpan`, `Guid`, `Uri`, `Version`, `char`, `StringBuilder`, `CultureInfo`, an enum | string |
| `XObject`, `XmlNode` | Its XML text |
| A delegate, reflection metadata, a `Task` or `ValueTask` | Refused with JM5923 `HostValueNotConvertible` |
| Any other object, struct, record, anonymous type, or tuple | object of its public readable properties, then its public fields |

A NaN or an infinity anywhere in the value is refused with JM5921 `HostNumberNaN` or JM5922 `HostNumberInfinite`. `FromObject` throws a `JyroRuntimeException` with the code; `WithInput` reports it through `Execute`. The conversion calls every public getter it reads and never calls `JsonSerializer`; a type whose shape is decided by `System.Text.Json` attributes or converters is serialized with `JsonSerializer.SerializeToElement` first and the element passed.

## ToObjectValue

| Jyro value | .NET value |
| --- | --- |
| null | `null` |
| boolean | `bool` |
| number, exact and integral within `long` | `long` |
| number, exact otherwise | `decimal` |
| number, inexact | `double` |
| string | `string` |
| array | `List<object>` |
| object | `Dictionary<string, object>` |
| function | The string `"<function>"` |

## Mutability

Arrays and objects are mutable and shared by reference, as they are in the language. A value graph given to a run is frozen as `Input` when `WithInput` is called, and remains frozen afterwards. A graph shared between concurrent runs is frozen first, or each run is given its own copy.

A walk of a value, including `EqualsValue`, `ToJson`, `ToObjectValue`, and `Freeze`, stops at a nesting depth of 128 with JM5910 `ValueNestingLimitExceeded`. A string that holds half of a surrogate pair has no JSON form, and `ToJson` throws for it.
