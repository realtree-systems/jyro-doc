# JyroNumber

A `JyroNumber` is exact or inexact. An exact number holds a value of the form *m* / 10^*s*, with *m* an integer of magnitude below 2^96 and *s* from 0 to 28, without rounding. Every other number is the nearest IEEE 754 double. Every number is finite.

| Constructor | Result |
| --- | --- |
| `new JyroNumber(int)`, `new JyroNumber(long)` | Exact |
| `new JyroNumber(decimal)` | Exact |
| `new JyroNumber(double)` | Inexact; throws `JyroRuntimeException` with JM5921 `HostNumberNaN` for NaN and JM5922 `HostNumberInfinite` for an infinity |

| Member | Description |
| --- | --- |
| `IsExact` | Whether the number is exact |
| `IsInteger` | Whether the value is a whole number |
| `DecimalValue` | The exact value; for an inexact number, the double converted to `decimal` |
| `Value`, `ToDouble()` | The nearest double; lossy for an exact value that no double holds |
| `ToInt32()`, `ToInt64()` | The integer, or `InvalidCast` when the value has a fraction or is out of range |
| `Negate()` | The negated number, with the same exactness |
| `static Compare(JyroNumber, JyroNumber)` | Two exact numbers by decimal value; otherwise by double value |

```csharp
using RealTree.Jyro;

var id = new JyroNumber(175928847299117063L);
var price = new JyroNumber(19.99m);
var ratio = new JyroNumber(0.1);

Console.WriteLine(id.IsExact);              // True
Console.WriteLine(id.ToJson());             // 175928847299117063
Console.WriteLine(price.DecimalValue);      // 19.99
Console.WriteLine(ratio.IsExact);           // False
Console.WriteLine(price.ToObjectValue());   // 19.99, a decimal
Console.WriteLine(id.ToObjectValue());      // 175928847299117063, a long
```

`JyroNumber.Equals` and `GetHashCode` compare mathematical values, so an exact `1` equals an inexact `1.0`, and a number can be a dictionary key. The language operator `==`, and `EqualsValue`, compare a mixed pair as doubles. In JSON output an exact number is written as its decimal, and an inexact integral number as an integer when it fits a `long`.

In a host function, `GetDecimal`, `GetInt64`, `GetInt32`, and `GetNonNegativeInt32` read a number exactly, and `GetNumber` reads it as a double. See [Defining a Function](/docs/host-api/host-functions/defining-a-function).
