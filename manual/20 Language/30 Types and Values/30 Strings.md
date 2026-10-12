# Strings

A string is a sequence of UTF-16 code units. A character outside the Basic Multilingual Plane occupies two code units. Strings are immutable: every operation that changes text returns a new string.

## Length and indexing

The standard library function `Length` returns the number of code units. Indexing a string with an integer returns the code unit at that position as a string of length one. A position outside the string, or a negative position, gives `null`.

```jyro
Output.length = Length("abc")
Output.first = "abc"[0]
Output.past = "abc"[5]
Output.units = Length("😀")
Output.joined = "😀"[0] & "😀"[1]
```

```json
{ "length": 3, "first": "a", "past": null, "units": 2, "joined": "😀" }
```

A code unit that is half of a surrogate pair is a valid string inside the script, but it has no UTF-8 form. Such a string in `Output` makes the conversion of the result to JSON fail.

## Concatenation

The `&` operator joins the string forms of its operands. A number or a boolean is converted to its string form, and `null` to the text `null`. An array, an object, or a function on either side is a runtime error. See [String Concatenation](/docs/language/expressions/string-concatenation).

```jyro
Output.a = "Total: " & 42
Output.b = "" & true & null
```

```json
{ "a": "Total: 42", "b": "truenull" }
```

The `+` operator does not accept a string operand.

## Comparison

Two strings are equal when they have the same code units. The relational operators compare strings code unit by code unit, by ordinal value.

```jyro
Output.a = "A" < "a"
Output.b = "app" < "apple"
Output.c = "abc" == "abc"
```

```json
{ "a": true, "b": true, "c": true }
```

## Iteration

A `foreach` loop over a string visits each code unit as a one-unit string. See [Foreach](/docs/language/statements/foreach).

## Case and white space

The standard library functions that change case and trim white space use the simple case mappings and the White_Space property of Unicode version 15. See the [String](/docs/standard-library/string/overview) functions.
