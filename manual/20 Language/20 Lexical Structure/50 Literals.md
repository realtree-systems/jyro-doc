# Literals

## Numbers

A decimal literal has the form of a JSON number without a sign: an integer part with no leading zero, an optional fraction, and an optional exponent.

```jyro
Output.a = 42
Output.b = 3.25
Output.c = 1.5e3
Output.d = 25e-1
```

```json
{ "a": 42, "b": 3.25, "c": 1500, "d": 2.5 }
```

A hexadecimal literal starts with `0x` or `0X`, and a binary literal with `0b` or `0B`. Both must fit in a signed 64-bit integer.

```jyro
Output.hex = 0xFF
Output.bin = 0b1010
```

```json
{ "hex": 255, "bin": 10 }
```

A literal has no sign. `-5` is the unary minus operator applied to the literal `5`. A leading `+` is a parse error.

A literal is an exact decimal when its value can be held without rounding: up to 28 decimal places and a significand below 2^96. Otherwise it is the nearest double. A literal with no finite double value, such as `1e400`, is refused with JM2200 `InvalidNumberFormat`.

The following forms are parse errors: `007`, `1.`, `.5`, `1e`, `1e+`, `12abc`, `0x`, `0b2`. A number token extends over every following letter, digit, and `.`, so `12abc` is one malformed token rather than `12` followed by `abc`.

## Strings

A string literal is enclosed in double quotes or single quotes. The two forms are equivalent.

```jyro
Output.a = "double"
Output.b = 'single'
Output.c = "it's"
```

```json
{ "a": "double", "b": "single", "c": "it's" }
```

The escape sequences are:

| Escape | Character |
| --- | --- |
| `\n` | Line feed, U+000A |
| `\r` | Carriage return, U+000D |
| `\t` | Tab, U+0009 |
| `\\` | Backslash |
| `\"` | Double quote |
| `\'` | Single quote |
| `\0` | Null, U+0000 |
| `\uXXXX` | The UTF-16 code unit with the four hexadecimal digits `XXXX` |

Any other escape, including a backslash before a line end, is refused with JM1201 `InvalidEscapeSequence`. A character outside the Basic Multilingual Plane is written as two `\u` escapes forming a surrogate pair. A surrogate that is not part of a pair, whether escaped or written directly, is refused with JM5918 `UnpairedSurrogate`.

```jyro
Output.a = "line 1\nline 2"
Output.b = "café"
Output.c = "😀"
```

```json
{ "a": "line 1\nline 2", "b": "café", "c": "😀" }
```

A line end may appear inside a string literal and is part of the string. A string with no closing quote is refused with JM1200 `UnterminatedString`. There is no string interpolation; values are joined with the `&` operator.

## Booleans and null

The literals are `true`, `false`, and `null`, in lowercase.

## Arrays

An array literal is a comma-separated list of expressions in square brackets. Elements may be of different types.

```jyro
Output.empty = []
Output.mixed = [1, "two", true, null, [3], { four: 4 }]
```

```json
{ "empty": [], "mixed": [1, "two", true, null, [3], { "four": 4 }] }
```

## Objects

An object literal is a comma-separated list of `key: value` pairs in braces. A key is an identifier, a reserved word, or a string literal. A string literal is required for a key that is not a valid identifier.

```jyro
Output.person = { name: "Ada", "first-name": "Ada", if: 1 }
Output.empty = {}
```

```json
{ "person": { "name": "Ada", "first-name": "Ada", "if": 1 }, "empty": {} }
```

Properties keep the order in which they are written. When a key is repeated, both values are evaluated, the last value is kept, and the property keeps the position of the first.

```jyro
Output.o = { a: 1, b: 2, a: 3 }
```

```json
{ "o": { "a": 3, "b": 2 } }
```

A trailing comma in an array literal, an object literal, or an argument list is a parse error.
