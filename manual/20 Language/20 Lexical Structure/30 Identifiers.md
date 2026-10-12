# Identifiers

An identifier names a variable, a parameter, a function, a union, a variant, a field, or a property.

An identifier starts with `_` or a Unicode letter, and continues with `_`, letters, or decimal digits. Letters are the characters of general category L and decimal digits those of category Nd, so letters and digits outside ASCII are accepted. A character outside the Basic Multilingual Plane counts as one character.

```jyro
var count = 1
var _private = 2
var naïve = 3
var 数 = 4
Output.sum = count + _private + naïve + 数
```

```json
{ "sum": 10 }
```

Identifiers are case-sensitive and are compared character for character. No Unicode normalization is applied: two spellings of the same text in different normalization forms are different identifiers. The same rule applies to function names, variant names, and the labels of named arguments.

```jyro
var value = 1
var Value = 2
Output.lower = value
Output.upper = Value
```

```json
{ "lower": 1, "upper": 2 }
```

A reserved keyword cannot be used as an identifier, except in the positions listed under [Keywords](/docs/language/lexical-structure/keywords). `_` on its own is an ordinary identifier. There is no limit on the length of an identifier.
