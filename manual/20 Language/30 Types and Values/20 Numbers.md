# Numbers

A number is either an exact decimal or an inexact double. An exact number holds up to 28 decimal places and a significand below 2^96 without rounding. An inexact number is an IEEE 754 double. A number is always finite: no operation produces NaN or an infinity.

## Exact and inexact numbers

Integer, hexadecimal, and binary literals are exact. A decimal literal is exact when its value fits the exact representation, and otherwise the nearest double. A number read from JSON input follows the same rule.

An arithmetic operation on two exact operands is performed in decimal arithmetic and gives an exact result. When the exact result cannot be held without rounding, as for `1 / 3`, or when it overflows the exact range, the operation is performed in double arithmetic and the result is inexact. An operation with an inexact operand gives an inexact result.

```jyro
Output.a = 0.1 + 0.2
Output.b = 0.1 * 3
Output.c = 10 / 4
Output.d = 1 / 3
Output.e = 1e30
```

```json
{ "a": 0.3, "b": 0.3, "c": 2.5, "d": 0.3333333333333333, "e": 1e+30 }
```

## Arithmetic

The operators `+`, `-`, `*`, `/`, and `%` take two numbers. Unary `-` takes one number and keeps its exactness. The result of `%` has the sign of the dividend.

```jyro
Output.a = 7 % 3
Output.b = -7 % 3
Output.c = 7 % -3
Output.d = 2 / 0.5
```

```json
{ "a": 1, "b": -1, "c": 1, "d": 4 }
```

Division by zero is a runtime error.

```jyro
Output.a = 1 / 0
```

```text
[JM5200] Ln 1, Col 12: Division by zero
```

```jyro
Output.a = 5 % 0
```

```text
[JM5201] Ln 1, Col 12: Modulo by zero
```

An operation whose result would be NaN is refused with JM5914 `NumberDomainError`, and one whose result would be an infinity with JM5915 `NumberOverflow`.

## Comparison

Two exact numbers compare by their decimal values. Any other pair compares by double value. Equality is a comparison, so an exact number and an inexact number of the same double value are equal.

```jyro
Output.a = 2 == 2.0
Output.b = 0.5 == 1 / 2
Output.c = 0.1 + 0.2 == 0.3
```

```json
{ "a": true, "b": true, "c": true }
```

## String form

The string form of a number is used by the `&` operator, by `ToString`, and by `exit` and `fail`. An exact number is written as a plain decimal with no exponent and no trailing zeros; zero is written as `0`. An inexact number is written as ECMAScript's `Number::toString` writes it.

```jyro
Output.a = "" & 1.50
Output.b = "" & 1e21
Output.c = "" & 1e30
Output.d = "" & 1 / 3
```

```json
{ "a": "1.5", "b": "1000000000000000000000", "c": "1e+30", "d": "0.3333333333333333" }
```

## JSON form

In JSON output an exact number is written as its decimal value. An inexact number with an integer value is written as an integer when it fits a signed 64-bit integer, and otherwise in double form.

```jyro
Output.a = 1 / 3 * 3
Output.b = 100000000000000000000000000000
```

```json
{ "a": 1, "b": 1e+29 }
```
