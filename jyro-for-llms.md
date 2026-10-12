# Jyro for LLMs

This document is written for a language model that must write Jyro programs from a description of a business problem. It is self-contained: everything needed to write a correct script is here, including the full standard library. Read it in order. Section 2 lists the assumptions carried over from other languages that produce incorrect Jyro; every one of them is an error in Jyro, not a style difference.

Conventions in this document: a Jyro example may start with a comment of the form `# input: {...}`, which is the JSON the host supplied as `Input` for that run. A `json` block after an example is the `Output` the example produces. A `text` block after an example is the diagnostic the example produces. Every example has been executed.

## 1. What Jyro is

Jyro is a scripting language that a .NET host application compiles and runs inside a sandbox. A script is a sequence of statements that run from the first to the last. There is no entry point, no module system, and no import.

- The script reads its data through `Input`, a read-only value the host supplies, usually a JSON object.
- The script writes its result to `Output`, which starts as an empty object `{}`. The value of `Output` when the script ends is the result the host receives.
- A script has no access to files, the network, the clock, the environment, or anything outside `Input`, except through functions the host chooses to provide.
- Values are the JSON types (null, boolean, number, string, array, object) plus lambda function values. Numbers are exact decimals: `0.1 + 0.2 == 0.3`.
- The standard library of 97 functions is available in practice in every host; assume it is present unless told otherwise. A host may add its own functions and will name them.
- Every run is bounded: by default 5 seconds, 10,000 statements, 1,000 loop iterations in total across all loops, and a call depth of 64. A standard library call counts as one statement however large its input, so **prefer standard library functions to explicit loops over data**.
- A runtime error stops the script at that statement. There is no exception handling. The host receives the diagnostic and the `Output` written so far.

A minimal script:

```jyro
# input: { "name": "Ada", "scores": [72, 88, 95] }
var average = Average(Input.scores)
Output.greeting = "Hello, " & Input.name
Output.average = average
Output.grade = average >= 80 ? "pass" : "fail"
```

```json
{ "greeting": "Hello, Ada", "average": 85, "grade": "pass" }
```

## 2. Assumptions to drop

Each row is a habit from JavaScript, Python, C#, or similar languages that produces incorrect Jyro.

| Habit from other languages | Jyro |
| --- | --- |
| `+` joins strings | `&` joins strings. `+` is numeric only; a string operand is a compile error. `"Total: " & total` |
| String interpolation `${x}`, `f"{x}"`, `$"{x}"` | None. Build text with `&`: `"Order " & id & " shipped"` |
| `//` or `/* */` comments | `#` to end of line is the only comment. `//` is a parse error |
| Semicolons end statements | No separators at all. Statements follow one another; line ends are optional |
| Braces `{ }` delimit blocks | Keywords: `if c then ... end`, `while c do ... end`, `for ... do ... end`, `func F() ... end`. Every block closes with `end` |
| `else if` | `elseif`, one word |
| `&&`, `\|\|`, `!` | `and`, `or`, `not` |
| `===`, `!==` | `==`, `!=` only. They compare by value, deeply, and never equate different types |
| `let`, `const`, `function`, `def` | `var` declares a variable. `func` declares a function |
| `x = y = 0`, assignment as an expression | Assignment is a statement only |
| `for (i = 0; i < n; i++)` | `for i in 0 to n do ... end`. The end value is exclusive |
| `for (x of arr)`, `for x in list` | `foreach x in arr do ... end` |
| `for (key in obj)` | `foreach key in Keys(obj) do ... end`. A `foreach` directly over an object yields its values, not its keys |
| `arr.length`, `str.length` | `Length(arr)`, `Length(str)`. No value has methods or properties of its own |
| `arr.push(x)`, `arr.append(x)` | `arr = Append(arr, x)`. Every library function returns a new value and changes nothing |
| `str.toUpperCase()`, `str.split(",")`, `arr.map(f)` | `ToUpper(str)`, `Split(str, ",")`, `Map(arr, f)`. Functions are called by name with the value as the first argument |
| `arr[-1]` for the last element | `null`. Use `Last(arr)` or `arr[Length(arr) - 1]` |
| `arr[5] = x` extends a short array | Runtime error. An index may replace an element or append at exactly `Length(arr)` |
| `obj.missing.deeper` throws | Gives `null`. Reading through a missing property or a null never fails; there is no `?.` |
| `x ?? y`, `x \|\| y` for defaults | `??` replaces `null` only. `0`, `false`, and `""` are kept. `or` returns an operand by truthiness |
| `[]` and `{}` are falsy | They are truthy. Only `null`, `false`, `0`, and `""` are falsy |
| `typeof x === "number"`, `isinstance` | `x is number`, `x is not string`. Types: `null`, `boolean`, `number`, `string`, `array`, `object`, `function`, `any` |
| `key in obj` | `HasProperty(obj, "key")` |
| `Math.floor(x)`, `Math.round(x)` | `Floor(x)`, `Round(x, 0)`. `Round` requires the number of decimals |
| Integer division `7 / 2 == 3` | `7 / 2` is `3.5`. Use `Floor(7 / 2)` |
| `throw`, `raise`, `try/catch` | `fail "message"` ends the run with an error. There is no catch |
| `return` at the top level | `exit`, or `exit "message"`. `return` is valid only inside `func` |
| Nested functions, closures over outer variables | `func` must be at the top level, and its body sees only its parameters, its locals, and `Input`. Pass values in as arguments |
| A function writes to a global result | Neither a `func` body nor a lambda body can reference `Output`. A `func` returns a value; a lambda reads variables |
| Arrow functions with a block body `x => { ... }` | A lambda body is a single expression: `x => x.total * 2` |
| Calling a stored lambda `f(1)` | Impossible. A lambda is only ever passed to a function that calls it, such as `Map`, `Where`, `Reduce` |
| Passing a function by name `Map(xs, Double)` | Not a value. Wrap it: `Map(xs, x => Double(x))` |
| `switch` with `break` and fall-through | `switch v do case 1, 2 then ... case 3 then ... default then ... end`. No fall-through, several values per case, `default` last |
| `print`, `console.log` | None in the language. Put what should be seen into `Output` |
| Mutating `Input` | `Input` is frozen. `Clone(Input.items)` gives a writable copy |
| Trailing commas `[1, 2,]` | Parse error |
| Variables named `end`, `to`, `by`, `in`, `do`, `case`, `default`, `match`, `string`, `number`, `object`, `array` | All reserved. Choose other names (`finish`, `target`, `step`, `kind`, `text`, `count`, `item`, `items`) |
| `null` arguments are errors | Passing `null` to an optional parameter means "use the default". For a string parameter `null` reads as `""`; for an array parameter as `[]` |
| Dates are objects | Dates are strings in the form `yyyy-MM-ddTHH:mm:ss.fffZ`, always UTC. `Now()`, `DateAdd`, `DateDiff`, `FormatDate` work on them |
| Regular expressions have flags like `/i` | No flags. Lower-case both sides for case-insensitive matching. Write `\\d` inside a string literal |
| Loops over any size of data | Loop iterations are capped (1,000 in total by default). Use `Map`, `Where`, `Reduce`, `Sum`, `GroupBy`, `Range` for data of unknown size |

Three of these, demonstrated:

```jyro
Output.a = "Total: " + 42
```

```text
[JM3301] Ln 1, Col 12: The '+' operator does not accept a string operand
```

```jyro
var f = x => x + 1
Output.a = f(1)
```

```text
[JM4100] Ln 2, Col 12: Undefined function 'f'
```

```jyro
var items = [1, 2]
items[5] = 3
```

```text
[JM5300] Ln 2, Col 1: array position 5 is out of bounds for a length of 2
```

## 3. A complete example

```jyro
# input: { "customer": { "name": "Ada Lovelace", "tier": "gold" }, "lines": [ { "sku": "A1", "qty": 2, "unitPrice": 9.99 }, { "sku": "B2", "qty": 1, "unitPrice": 24.5 }, { "sku": "C3", "qty": 0, "unitPrice": 3 } ] }
# Price an order: drop empty lines, total it, apply a tier discount.

var discountRate = Input.customer.tier == "gold" ? 0.1 : 0

var lines = Where(Input.lines, line => line.qty > 0)
var subtotal = Sum(Map(lines, line => line.qty * line.unitPrice))
var discount = Round(subtotal * discountRate, 2)

Output.customer = Input.customer.name
Output.lineCount = Length(lines)
Output.subtotal = subtotal
Output.discount = discount
Output.total = subtotal - discount
Output.summary = Output.customer & " owes " & Output.total
```

```json
{
  "customer": "Ada Lovelace",
  "lineCount": 2,
  "subtotal": 44.48,
  "discount": 4.45,
  "total": 40.03,
  "summary": "Ada Lovelace owes 40.03"
}
```

## 4. Lexical rules

**Comments.** `#` starts a comment that runs to the end of the line. There is no block comment.

**Identifiers.** Start with a letter or `_`, continue with letters, digits, or `_`. Case-sensitive. Standard library functions are PascalCase (`ToUpper`, `SortByField`); the exact spelling must be used.

**Reserved words** (38), which cannot be used as variable, parameter, or function names:

```text
and array boolean break by case continue default delete do downto else elseif end exit
fail false for foreach func if in is match not null number object or return string switch
then to true union var while
```

`any` and `function` are type names after `:` or `is` and are otherwise ordinary identifiers. A reserved word is allowed as a property name after `.` and as an object key: `Output.default = 1` and `{ for: 1 }` are valid.

**Numbers.** `42`, `3.25`, `1.5e3`, `0xFF`, `0b1010`. No leading zeros (`007`), no bare point (`.5`, `1.`), no sign in the literal (`-5` is unary minus applied to `5`).

**Strings.** `"text"` or `'text'`, equivalent. Escapes: `\n`, `\r`, `\t`, `\\`, `\"`, `\'`, `\0`, `\uXXXX` (exactly four hex digits). No other escape is valid, and there is no interpolation. A line end inside a literal is allowed and is part of the string.

**Booleans and null.** `true`, `false`, `null`, lowercase.

**Arrays and objects.** `[1, "a", null]`, `{ name: "Ada", "first-name": "Ada", if: 1 }`. Keys may be bare identifiers, reserved words, or strings. No trailing commas. A repeated key keeps the last value.

**Line ends.** Statements need no separator and may share a line. An expression may continue on the next line when that line starts with an operator, `.`, or `?`. A line that starts with `(`, `[`, `++`, or `--` begins a new statement, so an index or a call must open on the same line as the expression it applies to. The value after `return`, `exit`, or `fail` must start on the same line as the keyword.

```jyro
var a = 1 var b = 2
Output.sum = a
    + b
Output.name = { first: "Ada" }
    .first
```

```json
{ "sum": 3, "name": "Ada" }
```

## 5. Values

| Type | Literal | Notes |
| --- | --- | --- |
| null | `null` | The value of a missing property, an index out of range, an undeclared-initializer variable, and a function that returns nothing |
| boolean | `true`, `false` | |
| number | `42`, `3.5` | Exact decimal when it fits (28 decimal places, significand below 2^96), otherwise a double. Never NaN or infinite |
| string | `"text"` | A sequence of UTF-16 code units. Immutable |
| array | `[1, 2]` | Ordered, mutable, shared by reference |
| object | `{ a: 1 }` | Ordered string-keyed map, mutable, shared by reference. Properties keep insertion order |
| function | `x => x + 1` | A lambda value. Can be stored and passed, never called directly |

`TypeOf(value)` returns the type name as a string. `value is type` tests it.

### Truthiness

Falsy: `null`, `false`, `0`, `""`. Truthy: everything else, including `[]`, `{}`, `"0"`, `-1`, and every function. Conditions, `and`, `or`, `not`, and `Where` use truthiness.

### Equality and ordering

`==` and `!=` compare any two values and never fail. Different types are never equal: `0 == "0"`, `0 == false`, `null == 0`, `"" == null` are all `false`. Arrays and objects compare by content, deeply; property order does not matter. Functions compare by identity.

`<`, `<=`, `>`, `>=` compare two numbers or two strings (by code unit, so `"B" < "a"` and `"10" < "9"`). Any other pair is a runtime error.

```jyro
Output.a = [1, 2] == [1, 2.0]
Output.b = { x: 1, y: 2 } == { y: 2, x: 1 }
Output.c = "1" == 1
Output.d = null == false
Output.e = "apple" < "banana"
```

```json
{ "a": true, "b": true, "c": false, "d": false, "e": true }
```

### Numbers

- `+ - * / %` on numbers only. `/` gives a decimal: `7 / 2` is `3.5`. `%` keeps the sign of the dividend: `-7 % 3` is `-1`.
- Division or modulo by zero is a runtime error.
- Exact arithmetic: `0.1 + 0.2` is exactly `0.3`. A result that cannot be exact, such as `1 / 3`, becomes a double.
- The string form (used by `&` and `ToString`) of an exact number has no exponent and no trailing zeros: `1.50` prints as `1.5`.
- `Round(value, decimals)` needs the decimals argument. `Round(2.345, 2)` is `2.35` (half up). `Round(x, 0)` rounds to a whole number. `Floor` and `Ceiling` give integers.

### Strings

- `&` joins any non-container values: numbers and booleans become their text, `null` becomes `null`. An array or object operand is a runtime error; use `ToJson(value)` or `Join(array, ", ")`.
- `Length("abc")` is 3. `"abc"[0]` is `"a"`. An index out of range gives `null`.
- Comparison and `Contains`, `IndexOf`, `Replace`, `Split` are case-sensitive and exact. For case-insensitive work, apply `ToLower` to both sides.

### Arrays

- Read: `a[i]` for `i` from `0` to `Length(a) - 1`; any other index gives `null`; a fractional index is an error.
- Write: `a[i] = v` replaces an element, or appends when `i == Length(a)`. Any other index is a runtime error. The usual way to add is `a = Append(a, v)`.
- `delete` does not apply to arrays; use `RemoveAt`, `RemoveFirst`, `RemoveLast`.
- `foreach` visits a snapshot of the array taken when the loop starts.

### Objects

- Read: `o.name` or `o["name"]`. A missing property gives `null`; reading a property of a non-object gives `null`; a chain through nulls gives `null`.
- Write: `o.name = v` adds or replaces. The target must already be an object: `Output.order.total = 1` fails unless `Output.order` was assigned an object first.
- `delete o.name` removes a property. `Keys(o)`, `Values(o)`, `HasProperty(o, "name")`, `Merge([o1, o2])`, `Clone(o)`.
- Property order is insertion order, and `Keys` returns it.

### Reference semantics

Arrays and objects are shared by reference. Assigning, passing, and storing share the same value; a change through one reference is visible through all. `Clone(value)` makes a deep copy. Standard library functions return new containers but do not copy the elements inside them.

`Input` and everything reachable from it is frozen. A write through any reference to it is a runtime error; `Clone` first.

```jyro
# input: { "items": [ { "n": 1 } ] }
var items = Clone(Input.items)
items[0].n = 2
var alias = items
alias = Append(alias, { n: 3 })
Output.items = items
Output.alias = alias
Output.original = Input.items
```

```json
{ "items": [{ "n": 2 }], "alias": [{ "n": 2 }, { "n": 3 }], "original": [{ "n": 1 }] }
```

## 6. Variables and scope

```jyro
# fragment
var name                      # null
var name = expression
var name: number = expression # typed: every later assignment is converted to number or fails
```

- A variable must be declared before use, and once per scope. Blocks (`if`, loops, `case`, function bodies) are scopes; an inner block may shadow an outer variable.
- Assignment: `name = value`, `obj.prop = value`, `arr[i] = value`. Compound: `+= -= *= /= %= &=`. Increment: `x++`, `++x`, `x--`, `--x` (also on properties and elements).
- A typed variable converts on assignment: a string `"42"` into a `number` variable becomes `42`; `"abc"` is a runtime error. Types: `number`, `string`, `boolean`, `array`, `object`, `null`, `function`, `any`.
- `Input` and `Output` are the only predefined names. Neither can be declared or used as a parameter name.
- A `func` body is sealed: it sees its parameters, its own variables, `Input`, and other functions. It cannot see script-level variables and cannot reference `Output`.

## 7. Operators, tightest first

| Level | Operators | Associativity |
| --- | --- | --- |
| 1 | `f(args)`, `.name`, `[i]`, postfix `++` `--` | left |
| 2 | prefix `++` `--`, unary `-` | right |
| 3 | `*` `/` `%` | left |
| 4 | `+` `-` | left |
| 5 | `&` | left |
| 6 | `<` `<=` `>` `>=` | left |
| 7 | `==` `!=` | left |
| 8 | `is`, `is not` | not chainable |
| 9 | `not` | right |
| 10 | `and` | left |
| 11 | `or` | left |
| 12 | `??` | right |
| 13 | `c ? a : b` | right |
| 14 | `params => body` | body extends right |

Consequences: `"n=" & 1 + 2` is `"n=3"`; `not a == b` is `not (a == b)`; `x is number and x > 0` works without parentheses; `a ?? b or c` is `a ?? (b or c)`; `1 < 2 < 3` is an error (boolean compared with number). `and` and `or` short-circuit and return an operand: `null or "default"` is `"default"`, `"a" and "b"` is `"b"`. `??` short-circuits and only replaces `null`.

## 8. Statements

### if

```jyro
# input: { "score": 85 }
var grade
if Input.score >= 90 then
    grade = "A"
elseif Input.score >= 80 then
    grade = "B"
else
    grade = "C"
end
Output.grade = grade
```

```json
{ "grade": "B" }
```

### switch

Compares the subject with each case value by `==`, first match wins, no fall-through, `default` optional and last.

```jyro
# input: { "status": "created" }
switch Input.status do
    case "ok", "created" then
        Output.state = "success"
    case "error" then
        Output.state = "failure"
    default then
        Output.state = "unknown"
end
```

```json
{ "state": "success" }
```

### while, for, foreach

```jyro
var n = 1
var steps = 0
while n < 100 do
    n *= 2
    steps++
end
var up = []
for i in 0 to 5 do
    up = Append(up, i)
end
var down = []
for i in 10 downto 0 by 3 do
    down = Append(down, i)
end
var seen = []
foreach item in ["a", "b"] do
    seen = Append(seen, item)
end
var total = 0
foreach value in { x: 1, y: 2 } do
    total += value
end
Output.n = n
Output.steps = steps
Output.up = up
Output.down = down
Output.seen = seen
Output.total = total
```

```json
{ "n": 128, "steps": 7, "up": [0, 1, 2, 3, 4], "down": [10, 7, 4, 1], "seen": ["a", "b"], "total": 3 }
```

- `for i in start to finish [by step]`: `finish` is exclusive; `step` must be a positive integer; `downto` counts down. The loop variable is fresh each iteration and assigning to it does not change the sequence.
- `foreach x in collection`: arrays give elements, objects give values (use `Keys(obj)` for keys), strings give one-character strings. Any other value is a runtime error.
- `break` ends the innermost loop; `continue` starts its next iteration.
- Every iteration of every loop counts toward the run's loop limit (1,000 by default, in total). Use standard library functions for data of unknown size.

### return, exit, fail, delete

- `return [value]`: ends a `func` call. Only inside `func`. Bare `return` gives `null`.
- `exit [value]`: ends the whole run successfully with the current `Output`; the value, if any, becomes a message the host can read. Allowed anywhere, including inside a `func`.
- `fail [value]`: ends the run with an error whose message is the value's text. `Output` so far is returned with the error. Use it for business rule violations the host must see as failures.
- `delete obj.key` or `delete obj[key]`: removes a property. No effect when absent.

```jyro
# input: { "qty": -2 }
if Input.qty < 0 then
    fail "Quantity must not be negative: " & Input.qty
end
Output.ok = true
```

```text
[JM5004] Ln 3, Col 5: Quantity must not be negative: -2
```

## 9. Functions

```jyro
# fragment
func Name(required, typed: number, optional = "literal", typedOptional: number = -1)
    statements
    return value
end
```

Rules:

- Top level only: not inside `if`, a loop, or another `func`. Functions are visible throughout the script, including before their declaration, so order and mutual recursion are free.
- A default must be a literal (or `-` and a number literal). Required parameters come before optional ones. Passing `null` to an optional parameter selects its default.
- Typed parameters convert their argument (`"5"` to a `number` parameter becomes `5`).
- Calls use all positional or all named arguments, never mixed: `F(1, 2)` or `F(b: 2, a: 1)`.
- The body sees only its parameters, its locals, `Input`, and other functions. It cannot read script variables or touch `Output`. It can change the contents of an array or object passed to it, since those are shared by reference.
- A function name is not a value: it cannot be stored or passed. Wrap it in a lambda.
- A name cannot repeat a standard library or host function name (`Length`, `Sum`, `Log`, `Select`, ...), whether or not the library is enabled.
- Recursion is allowed; call depth is limited (64 by default).

```jyro
func Price(amount: number, taxRate: number = 0.1, currency: string = "USD")
    return currency & " " & Round(amount * (1 + taxRate), 2)
end
Output.a = Price(100)
Output.b = Price(100, 0.2)
Output.c = Price(amount: 50, currency: "EUR")
Output.d = Price(100, null, "GBP")
```

```json
{ "a": "USD 110", "b": "USD 120", "c": "EUR 55", "d": "GBP 110" }
```

```jyro
var rate = 0.2
func Tax(amount)
    return amount * rate
end
```

```text
[JM3100] Ln 3, Col 21: Undeclared variable 'rate'
```

## 10. Lambdas

Forms: `x => expr`, `(x) => expr`, `(a, b) => expr`, `() => expr`. The body is one expression, which may be a conditional, a `match` expression, an object or array literal, or a call. A lambda captures the variables in scope where it is written and reads their current values when it runs. It can read `Input`, never `Output`.

A lambda is passed to a function that calls it. The standard library functions that take one, and the arguments they pass:

| Function | Lambda parameters | Returns |
| --- | --- | --- |
| `Map(items, f)` | `(item)` | array of results |
| `Where(items, f)` | `(item)` | elements where the result is truthy |
| `Find(items, f)` | `(item)` | first element where the result is truthy, or `null` |
| `Any(items, f)`, `All(items, f)` | `(item)` | boolean |
| `SortBy(items, f)` | `(item)` | elements ordered by the key `f` returns |
| `Each(items, f)` | `(item)` | `null`; used for side effects such as `item.flag = true` through `item.count++` |
| `Reduce(items, f, initial)` | `(accumulator, item)` | the last accumulator |

The lambda must declare exactly the number of parameters the function passes. A lambda cannot be called by the script. To reuse logic, declare a `func` and call it from a lambda: `Map(items, x => Score(x))`.

```jyro
# input: { "orders": [ { "id": 1, "total": 50, "tags": ["new"] }, { "id": 2, "total": 150, "tags": [] } ] }
var threshold = 100
Output.large = Map(Where(Input.orders, o => o.total > threshold), o => o.id)
Output.sum = Reduce(Input.orders, (acc, o) => acc + o.total, 0)
Output.first = Find(Input.orders, o => Length(o.tags) > 0).id
Output.sorted = Map(SortBy(Input.orders, o => -o.total), o => o.id)
Output.labels = Map(Input.orders, o => o.total >= 100 ? "big" : "small")
```

```json
{ "large": [2], "sum": 200, "first": 1, "sorted": [2, 1], "labels": ["small", "big"] }
```

## 11. Unions and match

A union declares a closed set of variants, each with zero or more fields. A variant is constructed by calling its name; the value is an object with a `_variant` property and one property per field. `match` must cover every variant and has no `default`. `match` exists as a statement and as an expression.

```jyro
union Payment
    Card(last4: string)
    Transfer(reference: string)
    Cash
end
func Describe(p)
    return match p do
        case Card(last4) then "card ending " & last4
        case Transfer(ref) then "transfer " & ref
        case Cash then "cash"
    end
end
Output.a = Describe(Card(4242))
Output.b = Describe(Cash())
Output.value = Transfer("T-1")
match Card("0001") do
    case Card(digits) then
        Output.digits = digits
    case Transfer(ref) then
        Output.digits = null
    case Cash then
        Output.digits = null
end
```

```json
{
  "a": "card ending 4242",
  "b": "cash",
  "value": { "_variant": "Transfer", "reference": "T-1" },
  "digits": "0001"
}
```

Unions are declared at the top level only. A zero-field variant is constructed with parentheses: `Cash()`. Unions are optional; most business scripts use plain objects and `switch`.

## 12. Standard library

Conventions that hold across the library:

- Every function returns a new value; no argument is modified. Assign the result: `items = Append(items, x)`.
- A string parameter accepts a number or a boolean (read as its text) and `null` (read as `""`). An array parameter accepts `null` as `[]`, and an object parameter `null` as `{}`. A number parameter does not accept `null`.
- An optional parameter, shown in brackets, takes its default when omitted or given `null`.
- Named arguments use the parameter names shown: `Substring(text: s, start: 1)`.
- Functions that search return `null` or `-1` when nothing is found, never an error: `First([])`, `Find(...)`, `RegexMatch(...)`, `ToNumber("x")` give `null`; `IndexOf` gives `-1`. Write `ToNumber(s) ?? 0` for a default.
- `Min`, `Max`, `Average`, `Median`, `Mode` give `null` for an empty array; `Sum` gives `0`. Non-number elements, `null` included, are ignored by the aggregates.
- Positions and lengths count from 0. `Slice(items, start, [finish])` and `Range(start, finish, [step])` exclude `finish`. `Substring(text, start, [length])` takes a length, not an end. `RandomInt(min, max)` excludes `max`.
- `Round(value, decimals, [mode])` requires `decimals`; `mode` is `"halfUp"` (default), `"halfEven"`, or `"halfDown"`.
- `SortByField(items, fieldName, direction)` requires `direction`, `"asc"` or `"desc"`. `Sort` and `SortBy` sort ascending; negate a numeric key for descending.
- The `ByField` query functions take an operator string: `"=="`, `"!="`, `"<"`, `"<="`, `">"`, `">="`.
- `Merge` and `Coalesce` take one array argument: `Merge([a, b])`, `Coalesce([x, y, "default"])`.
- Dates are strings. Input in the form `yyyy-MM-dd`, `yyyy-MM-ddTHH:mm:ss`, with an optional fraction and `Z` or `+HH:mm` offset. Output is always `yyyy-MM-ddTHH:mm:ss.fffZ` in UTC. Units for `DateAdd` and `DateDiff`: `day`, `week`, `month`, `year`, `hour`, `minute`, `second`, singular or plural. Parts for `DatePart`: `year`, `month`, `day`, `hour`, `minute`, `second`, `dayofweek` (0 is Sunday), `dayofyear`. `FormatDate` tokens: `yyyy MM M dd d HH H mm m ss s fff`, literal text in single quotes.
- Regular expressions are ECMAScript syntax with no flags, in a string literal, so `\d` is written `"\\d"`. Groups `( )`, named groups `(?<name> )`, lookahead, lookbehind, and lazy quantifiers are supported; `\p{}` and flags are not.
- `Now`, `Today`, `NewGuid`, `RandomInt`, `RandomChoice`, `RandomString` are non-deterministic; `Sleep` blocks. Every other function is pure.

The complete library, by category. The signature shows each parameter with its type; brackets mark an optional parameter.

<!-- stdlib-catalog:start -->
<!-- generated by docs/tools/Generate.fsx in the jyro repository; do not edit by hand -->

### String functions

- **`ToUpper(text: string)` -> string**. Converts text to upper case.
  Returns: The text with each letter in upper case, one character for one character, by the simple case mappings of Unicode version 15. Never null.
  Example: `ToUpper("ok")` returns `"OK"`.
- **`ToLower(text: string)` -> string**. Converts text to lower case.
  Returns: The text with each letter in lower case, one character for one character, by the simple case mappings of Unicode version 15. Never null.
  Example: `ToLower("Hello")` returns `"hello"`.
- **`Trim(text: string)` -> string**. Removes white space from both ends of text.
  Returns: The text without the white space at its start and its end; white space inside it is kept. White space is every character that Unicode version 15 gives the White_Space property. Never null.
  Example: `Trim("  a b  ")` returns `"a b"`.
- **`Replace(text: string, oldValue: string, newValue: string)` -> string**. Replaces every occurrence of one piece of text with another.
  Returns: The text with each occurrence of oldValue, found from left to right without overlapping, replaced by newValue; the text unchanged when oldValue does not occur in it. Never null.
  Example: `Replace("a-b-c", "-", "+")` returns `"a+b+c"`.
- **`Contains(source: any, value: any)` -> boolean**. Tells whether a string holds a piece of text, or an array holds a value.
  Returns: True when source holds value, false otherwise. Never null.
  Example: `Contains("haystack", "st")` returns true; `Contains([1, 2, 3], 2)` returns true.
- **`StartsWith(text: string, prefix: string)` -> boolean**. Tells whether text begins with a given prefix.
  Returns: True when text begins with prefix, false otherwise. Never null.
  Example: `StartsWith("filename.txt", "file")` returns true.
- **`EndsWith(text: string, suffix: string)` -> boolean**. Tells whether text ends with a given suffix.
  Returns: True when text ends with suffix, false otherwise. Never null.
  Example: `EndsWith("filename.txt", ".txt")` returns true.
- **`Split(text: string, separator: string)` -> array**. Splits text into an array of parts at each occurrence of a separator.
  Returns: An array of the strings between the separators, in order. Separators next to each other, or at the start or the end of text, give empty strings, so the array has one more element than there are separators; text with no separator gives an array of text alone. Never null.
  Example: `Split("a,b,,c", ",")` returns `["a", "b", "", "c"]`.
- **`Join(values: array, separator: string)` -> string**. Joins the elements of an array into one string, with a separator between each two.
  Returns: The elements in order with separator between each two; the empty string for an empty array. Never null.
  Example: `Join(["a", 1, true], "-")` returns `"a-1-true"`.
- **`Substring(text: string, start: number, [length]: number)` -> string**. Takes part of text, from a start position, of a given length or to the end.
  Returns: The part of text that begins at start. Never null.
  Example: `Substring("abcdef", 2, 3)` returns `"cde"`; `Substring("abcdef", 4)` returns `"ef"`.
- **`ToNumber(text: string)` -> any**. Reads a number from text written as a JSON number.
  Returns: The number the text writes, or null when the text is not a JSON number, for example "1,234", "+5", ".5", "01", "Infinity" or the empty string, and when it writes a number too large to represent, such as "1e400".
  Example: `ToNumber(" 12.5 ")` returns 12.5; `ToNumber("abc") ?? 0` returns 0.
- **`PadLeft(text: string, length: number, [padding]: string)` -> string**. Pads text on the left to a given length.
  Returns: The text with padding added before it until it is length long, or the text unchanged when it is already that long. Never null.
  Example: `PadLeft("42", 5, "0")` returns `"00042"`.
- **`PadRight(text: string, length: number, [padding]: string)` -> string**. Pads text on the right to a given length.
  Returns: The text with padding added after it until it is length long, or the text unchanged when it is already that long. Never null.
  Example: `PadRight("ab", 4, ".")` returns `"ab.."`.
- **`RegexTest(text: string, pattern: string)` -> boolean**. Tells whether a regular expression matches anywhere in text.
  Returns: True when pattern matches some part of text, false otherwise. Never null.
  Example: `RegexTest("order-123", "[0-9]+")` returns true.
- **`RegexMatch(text: string, pattern: string)` -> string**. Finds the first match of a regular expression in text.
  Returns: The text of the first, leftmost match, which may be the empty string; null when pattern matches nowhere in text.
  Example: `RegexMatch("order-123", "[0-9]+")` returns `"123"`.
- **`RegexMatchAll(text: string, pattern: string)` -> array**. Finds every match of a regular expression in text.
  Returns: An array of the text of each match, from left to right, the matches not overlapping; after an empty match the search goes on from the next character. The empty array when there is no match. Never null.
  Example: `RegexMatchAll("a1b22c333", "[0-9]+")` returns `["1", "22", "333"]`.
- **`RegexMatchDetail(text: string, pattern: string)` -> object**. Finds the first match of a regular expression in text, with its position and its capture groups.
  Returns: An object for the first, leftmost match, with three properties: match, the text matched; index, the position in text where it begins, counted from 0 in UTF-16 code units; and groups, an array of the text of each capture group in order, null for a group that took no part in the match. Null when pattern matches nowhere in text.
  Example: `RegexMatchDetail("id: 42", "([a-z]+): ([0-9]+)")` returns `{"match": "id: 42", "index": 0, "groups": ["id", "42"]}`.
- **`RandomString(length: number, [characterSet]: string)` -> string**. Makes a string of random characters.
  Returns: A string of length characters, each drawn at random from characterSet. Never null.
  Example: `RandomString(8, "0123456789")` returns eight random digits, such as `"40912785"`.
  Effect: non-deterministic.

### Array functions

- **`Length(source: any)` -> number**. Gives the size of a string, an array or an object.
  Returns: For a string, its length in UTF-16 code units; for an array, its number of elements; for an object, its number of properties; for null, 0. A whole number, exact. Never null.
  Example: `Length("abc")` returns 3; `Length([1, 2])` returns 2; `Length(null)` returns 0.
- **`Append(items: array, value: any)` -> array**. Adds a value to the end of an array.
  Returns: A new array holding the elements of items in order, followed by value. Never null.
  Example: `Append([1, 2], 3)` returns `[1, 2, 3]`.
- **`Prepend(items: array, value: any)` -> array**. Adds a value to the start of an array.
  Returns: A new array holding value, followed by the elements of items in order. Never null.
  Example: `Prepend([2, 3], 1)` returns `[1, 2, 3]`.
- **`First(items: array)` -> any**. Gives the first element of an array.
  Returns: The element at position 0. Null when items is empty or null, and when its first element is itself null.
  Example: `First([7, 8, 9])` returns 7; `First([])` returns null.
- **`Last(items: array)` -> any**. Gives the last element of an array.
  Returns: The element at the highest position. Null when items is empty or null, and when its last element is itself null.
  Example: `Last([7, 8, 9])` returns 9; `Last([])` returns null.
- **`IndexOf(source: any, value: any)` -> number**. Finds the position of the first occurrence of a piece of text in a string, or of a value in an array.
  Returns: The position of the first match, counted from 0: in UTF-16 code units for a string, in elements for an array. -1 when value is not found, or when source is null. A whole number, exact. Never null.
  Example: `IndexOf("banana", "na")` returns 2; `IndexOf([10, 20, 30], 30)` returns 2; `IndexOf([1, 2], 5)` returns -1.
- **`Reverse(items: array)` -> array**. Reverses the order of the elements of an array.
  Returns: A new array holding the elements of items from the last to the first. Never null.
  Example: `Reverse([1, 2, 3])` returns `[3, 2, 1]`.
- **`Slice(items: array, start: number, [finish]: number)` -> array**. Takes the elements of an array between two positions.
  Returns: A new array holding the elements of items from start up to, but not including, finish, in order. Never null.
  Example: `Slice([10, 20, 30, 40], 1, 3)` returns `[20, 30]`; `Slice([10, 20, 30, 40], 2)` returns `[30, 40]`.
- **`Concatenate(first: array, second: array)` -> array**. Joins two arrays into one.
  Returns: A new array holding the elements of first in order, followed by the elements of second in order. Never null.
  Example: `Concatenate([1, 2], [3])` returns `[1, 2, 3]`.
- **`Distinct(items: array)` -> array**. Removes the repeated elements of an array, keeping the first of each.
  Returns: A new array holding each element of items that is not equal, by the `==` operator, to any element before it, in their original order. Never null.
  Example: `Distinct([1, 2, 1, 3, 2])` returns `[1, 2, 3]`.
- **`Sort(items: array)` -> array**. Sorts the elements of an array in ascending order.
  Returns: A new array holding the elements of items in ascending order. Never null.
  Example: `Sort([3, 1, 2])` returns `[1, 2, 3]`; `Sort(["b", null, 1])` returns `[null, 1, "b"]`.
- **`Flatten(items: array)` -> array**. Flattens an array of arrays, at every level of nesting, into one array.
  Returns: A new array holding every element of items that is not an array, in order, with each nested array replaced by its own elements, flattened the same way. Objects, and null elements, are kept as they are. Never null.
  Example: `Flatten([1, [2, [3, [4]]]])` returns `[1, 2, 3, 4]`.
- **`Range(start: number, finish: number, [step]: number)` -> array**. Makes an array of numbers that count from a start toward a finish by a step.
  Returns: A new array of the numbers start, start + step, start + 2 * step and so on, while each is below finish for a positive step, or above finish for a negative one. The empty array when start is already at or past finish in that direction. Never null.
  Example: `Range(0, 5)` returns `[0, 1, 2, 3, 4]`; `Range(10, 0, -3)` returns `[10, 7, 4, 1]`.
- **`Skip(items: array, count: number)` -> array**. Drops a number of elements from the start of an array.
  Returns: A new array holding the elements of items after the first count, in order. Never null.
  Example: `Skip([1, 2, 3, 4], 2)` returns `[3, 4]`.
- **`Insert(items: array, index: number, value: any)` -> array**. Inserts a value into an array at a given position.
  Returns: A new array holding the elements of items with value placed at index, the elements from index onward each moved one position later. Never null.
  Example: `Insert([1, 3], 1, 2)` returns `[1, 2, 3]`.
- **`RemoveAt(items: array, index: number)` -> array**. Removes the element at a given position of an array.
  Returns: A new array holding the elements of items without the one at index, in order. Never null.
  Example: `RemoveAt([1, 2, 3], 1)` returns `[1, 3]`.
- **`RemoveFirst(items: array)` -> array**. Removes the first element of an array.
  Returns: A new array holding every element of items but the first, in order; the empty array when items is empty or null. Never null.
  Example: `RemoveFirst([1, 2, 3])` returns `[2, 3]`.
- **`RemoveLast(items: array)` -> array**. Removes the last element of an array.
  Returns: A new array holding every element of items but the last, in order; the empty array when items is empty or null. Never null.
  Example: `RemoveLast([1, 2, 3])` returns `[1, 2]`.
- **`FlattenOnce(items: array)` -> array**. Flattens one level of nesting in an array of arrays.
  Returns: A new array in which each element of items that is an array is replaced by its own elements, which are kept as they are even when they are arrays, and every other element, null included, is kept as it is, all in order. Never null.
  Example: `FlattenOnce([1, [2, [3]]])` returns `[1, 2, [3]]`.
- **`RandomChoice(items: array)` -> any**. Picks an element of an array at random.
  Returns: One element of items. Null when items is empty or null, and when the element picked is itself null.
  Example: `RandomChoice(["red", "green", "blue"])` returns one of the three strings.
  Effect: non-deterministic.
- **`SortByField(items: array, fieldName: string, direction: string)` -> array**. Sorts an array of objects by the value of one of their properties.
  Returns: A new array holding the elements of items ordered by the value of fieldName, in the given direction. Never null.
  Example: `SortByField([{"n": 2}, {"n": 1}], "n", "asc")` returns `[{"n": 1}, {"n": 2}]`.
- **`GroupBy(items: array, fieldName: string)` -> object**. Groups the objects of an array by the value of one of their properties.
  Returns: A new object with one property per group. Its key is the string form of the property's value, and the text null for an object whose value is null or that lacks the property. Its value is an array of the objects in that group, in their original order. The empty object when items holds no object. Never null.
  Example: `GroupBy([{"t": "a"}, {"t": "b"}, {"t": "a"}], "t")` returns `{"a": [{"t": "a"}, {"t": "a"}], "b": [{"t": "b"}]}`.
- **`SelectMany(items: array, fieldName: string)` -> array**. Collects the elements of an array property of each object in an array into one array.
  Returns: A new array holding the elements of the fieldName array of each object, in the order of the objects and then of each array's elements. Nested arrays among those elements are kept as they are. Never null.
  Example: `SelectMany([{"tags": ["a", "b"]}, {"tags": ["c"]}], "tags")` returns `["a", "b", "c"]`.

### Math functions

- **`Absolute(value: number)` -> number**. Gives the absolute value of a number.
  Returns: The number without its sign: value itself when it is zero or more, and its negation when it is below zero. The result is exact when value is exact. Never null.
  Example: `Absolute(-3.5)` returns `3.5`.
- **`Floor(value: number)` -> number**. Rounds a number down to the nearest integer.
  Returns: The greatest integer that is not above value, so a negative number goes away from zero. The result is exact when value is exact. Never null.
  Example: `Floor(2.7)` returns `2`; `Floor(-2.1)` returns `-3`.
- **`Ceiling(value: number)` -> number**. Rounds a number up to the nearest integer.
  Returns: The least integer that is not below value, so a negative number goes toward zero. The result is exact when value is exact. Never null.
  Example: `Ceiling(2.1)` returns `3`; `Ceiling(-2.7)` returns `-2`.
- **`Round(value: number, decimals: number, [mode]: string)` -> number**. Rounds a number to a given number of digits after the point.
  Returns: The value rounded to decimals digits after the point. When value already has no more digits than that, it is returned unchanged. The result is exact when value is exact and the result can be held exactly; otherwise it is the nearest floating-point value. A result too large to hold is a runtime error. Never null.
  Example: `Round(1.25, 1)` returns `1.3`; `Round(2.5, 0, "halfEven")` returns `2`; `Round(1250, -2)` returns `1300`.
- **`Min(values: array)` -> number**. Finds the least number in an array.
  Returns: The least of the numbers, returned as it is in the array; of several equal least numbers, the first. Null when the array holds no numbers: when it is empty, null, or holds only null elements.
  Example: `Min([3, 1, null, 2])` returns `1`; `Min([])` returns null.
- **`Max(values: array)` -> number**. Finds the greatest number in an array.
  Returns: The greatest of the numbers, returned as it is in the array; of several equal greatest numbers, the first. Null when the array holds no numbers: when it is empty, null, or holds only null elements.
  Example: `Max([3, 1, null, 2])` returns `3`; `Max([])` returns null.
- **`Power(value: number, exponent: number)` -> number**. Raises a number to a power.
  Returns: value raised to exponent; any number to the power 0 is 1. The result is exact when value is exact, exponent is an exact integer, and the result can be held exactly; otherwise the result is rounded to the nearest floating-point value. A result too large to hold, such as `Power(0, -1)`, is a runtime error. Never null.
  Example: `Power(2, 10)` returns `1024`; `Power(9, 0.5)` returns `3`.
- **`SquareRoot(value: number)` -> number**. Gives the square root of a number.
  Returns: The non-negative square root of value. The result is exact when value is exact and its square root can be held exactly, as for `2.25`; otherwise it is rounded to the nearest floating-point value. Never null.
  Example: `SquareRoot(2.25)` returns `1.5`.
- **`Log(value: number, [base]: number)` -> number**. Gives the logarithm of a number, natural or to a given base.
  Returns: The logarithm of value to base. When value and base are both exact and value is base raised to an integer power, that integer is returned exactly; otherwise the result is rounded to the nearest floating-point value. Never null.
  Example: `Log(1000, 10)` returns `3`; `Log(1)` returns `0`.
- **`Sum(values: array)` -> number**. Adds up the numbers in an array.
  Returns: The total of the numbers; 0 when the array holds none. The total is exact when every number is exact and the total can be held exactly; otherwise the numbers are added in order as floating-point values. A total too large to hold, such as `Sum([1e300, 1e300])`, is a runtime error. Never null.
  Example: `Sum([1, 2, null, 4])` returns `7`; `Sum([])` returns `0`.
- **`Average(values: array)` -> number**. Gives the mean of the numbers in an array.
  Returns: The total of the numbers divided by how many there are. The mean is exact when every number is exact and the mean can be held exactly; otherwise it is computed with floating-point values, and a finite mean is still given when the total itself would be too large to hold. Null when the array holds no numbers: when it is empty, null, or holds only null elements.
  Example: `Average([1, 2, null, 6])` returns `3`; `Average([])` returns null.
- **`Clamp(value: number, min: number, max: number)` -> number**. Limits a number to a range.
  Returns: min when value is below min, max when value is above max, and value otherwise; the number chosen is returned as it was given. Never null.
  Example: `Clamp(15, 0, 10)` returns `10`; `Clamp(5, 0, 10)` returns `5`.
- **`Median(values: array)` -> number**. Gives the middle value of the numbers in an array.
  Returns: With the numbers in ascending order, the middle one, returned as it is in the array, when there is an odd count; the mean of the two middle ones when there is an even count. That mean is exact when both are exact and it can be held exactly, and otherwise is computed with floating-point values. Null when the array holds no numbers: when it is empty, null, or holds only null elements.
  Example: `Median([3, 1, 2])` returns `2`; `Median([4, 1, 3, 2])` returns `2.5`.
- **`Mode(values: array)` -> number**. Finds the number that occurs most often in an array.
  Returns: The number that occurs most often, as it first appears in the array; when several occur equally often, the one that appears first. Null when the array holds no numbers: when it is empty, null, or holds only null elements.
  Example: `Mode([1, 2, 2, 3, 3])` returns `2`; `Mode([])` returns null.
- **`RandomInt(min: number, max: number)` -> number**. Returns a random integer from a range.
  Returns: An integer at least min and below max, every one of them equally likely. The integer is exact. Never null.
  Example: `RandomInt(1, 7)` returns one of 1, 2, 3, 4, 5 or 6.
  Effect: non-deterministic.

### Date and Time functions

- **`Now()` -> string**. Gives the current date and time in UTC.
  Returns: The current instant as an ISO 8601 string in UTC, to the millisecond, in the form `yyyy-MM-ddTHH:mm:ss.fffZ`, for example `"2024-01-15T09:30:00.000Z"`. Never null.
  Example: `Now()` returns a string such as `"2024-01-15T09:30:00.000Z"`.
  Effect: non-deterministic.
- **`Today()` -> string**. Gives the current date in UTC.
  Returns: The current date in UTC as a string in the form `yyyy-MM-dd`, with no time, for example `"2024-01-15"`. Never null.
  Example: `Today()` returns a string such as `"2024-01-15"`.
  Effect: non-deterministic.
- **`ParseDate(text: string)` -> string**. Reads a date written as an ISO 8601 string and gives it in the standard UTC form.
  Returns: The instant text gives, in UTC to the millisecond, in the form `yyyy-MM-ddTHH:mm:ss.fffZ`; null when text is not a date in one of the accepted forms.
  Example: `ParseDate("2024-01-15T10:30:00+02:00")` returns `"2024-01-15T08:30:00.000Z"`; `ParseDate("15/01/2024")` returns null.
- **`FormatDate(date: string, format: string)` -> string**. Writes a date as text, following a format pattern.
  Returns: The text the pattern gives for date. Never null.
  Example: `FormatDate("2024-01-15T09:05:00Z", "dd/MM/yyyy HH:mm")` returns `"15/01/2024 09:05"`; `FormatDate("2024-01-15", "'Day' d")` returns `"Day 15"`.
- **`DateAdd(date: string, amount: number, unit: string)` -> string**. Adds an amount of a unit of time to a date.
  Returns: The date that results, in UTC to the millisecond, in the form `yyyy-MM-ddTHH:mm:ss.fffZ`. A result before `0001-01-01T00:00:00.000Z` or after `9999-12-31T23:59:59.999Z` is a runtime error. Never null.
  Example: `DateAdd("2024-01-15", 10, "days")` returns `"2024-01-25T00:00:00.000Z"`; `DateAdd("2024-01-15T12:00:00Z", -2, "hours")` returns `"2024-01-15T10:00:00.000Z"`.
- **`DateDiff(startDate: string, endDate: string, unit: string)` -> number**. Measures the time from one date to another, in a given unit.
  Returns: The number of units from startDate to endDate. Never null.
  Example: `DateDiff("2024-01-01", "2024-01-15", "days")` returns 14; `DateDiff("2024-01-31", "2024-03-30", "months")` returns 1.
- **`DatePart(date: string, part: string)` -> number**. Gives one part of a date, such as its year or its day of the week, as a number.
  Returns: The part asked for, as a whole number. Never null.
  Example: `DatePart("2024-01-15T09:30:00Z", "hour")` returns 9; `DatePart("2024-01-15", "dayofweek")` returns 1.

### Utility functions

- **`TypeOf(value: any)` -> string**. Gives the name of a value's type.
  Returns: One of the strings `"null"`, `"boolean"`, `"number"`, `"string"`, `"array"`, `"object"` or `"function"`. Never null.
  Example: `TypeOf([1, 2])` returns `"array"`; `TypeOf(null)` returns `"null"`.
- **`Coalesce(values: array)` -> any**. Gives the first element of an array that is not null.
  Returns: The first element that is not null, as it is. Null when the array is empty or every element in it is null.
  Example: `Coalesce([null, "", "x"])` returns `""`; `Coalesce([null, null])` returns null.
- **`ToString(value: any)` -> string**. Converts a value to its string form.
  Returns: The string form of value. Never null.
  Example: `ToString(42)` returns `"42"`; `ToString([1, "a"])` returns `"[1,\"a\"]"`.
- **`ToBoolean(value: any)` -> boolean**. Converts a value to a boolean by the language's truthiness rule.
  Returns: False for null, false, the number zero and the empty string; true for every other value, an empty array, an empty object and a function included. Never null.
  Example: `ToBoolean("")` returns false; `ToBoolean([])` returns true.
- **`Keys(source: object)` -> array**. Lists the property names of an object.
  Returns: An array of the property names, as strings, in the order the object holds them. The empty array for an empty object. Never null.
  Example: `Keys({"a": 1, "b": 2})` returns `["a", "b"]`.
- **`Values(source: object)` -> array**. Lists the property values of an object.
  Returns: An array of the property values, as they are, in the order the object holds them, so in the same order as `Keys` gives the names. The empty array for an empty object. Never null.
  Example: `Values({"a": 1, "b": 2})` returns `[1, 2]`.
- **`HasProperty(source: object, name: string)` -> boolean**. Tells whether an object has a property of a given name.
  Returns: True when source has a property named name, whatever its value, null included; false otherwise. Never null.
  Example: `HasProperty({"a": null}, "a")` returns true; `HasProperty({"a": 1}, "A")` returns false.
- **`Merge(values: array)` -> object**. Combines the properties of several objects into one new object.
  Returns: A new object holding every property of every object in values. When more than one object has a property of the same name, the value of the last of them is kept. The objects in values are not changed. The empty object when values holds no object. Never null.
  Example: `Merge([{"a": 1, "b": 2}, {"b": 3}])` returns `{"a": 1, "b": 3}`.
- **`Clone(value: any)` -> any**. Makes a deep copy of a value.
  Returns: A copy equal to value. An array or an object that appears more than once inside value is copied once for each place it appears, so the copy shares no container with value or within itself. Null when value is null; never null otherwise.
  Example: `copy = Clone(data)` followed by `copy.name = "x"` leaves `data.name` unchanged.
- **`NewGuid()` -> string**. Makes a new random UUID string.
  Returns: A UUID of 36 characters in lower case: 32 hexadecimal digits in groups of 8, 4, 4, 4 and 12, joined by hyphens. Never null.
  Example: `NewGuid()` returns a string such as `"3f2b8c1e-9a4d-4e6f-8b2a-1c5d7e9f0a3b"`.
  Effect: non-deterministic.
- **`Base64Encode(text: string)` -> string**. Encodes text as Base64.
  Returns: The Base64 text of the UTF-8 bytes of text, in the standard alphabet with `=` padding and no line breaks; the empty string for empty text. `Base64Decode` gives the text back. Never null.
  Example: `Base64Encode("Hello")` returns `"SGVsbG8="`.
- **`Base64Decode(text: string)` -> string**. Decodes Base64 text back to the text it encodes.
  Returns: The text whose UTF-8 bytes text encodes. Null when text is not valid Base64, that is when, without the ignored characters, its length is not a multiple of four, it holds a character outside the alphabet, or `=` appears anywhere but in its last one or two places; null too when the decoded bytes are not valid UTF-8. A result that would pass the host's maximum value size is a runtime error.
  Example: `Base64Decode("SGVsbG8=")` returns `"Hello"`; `Base64Decode("abc")` returns null, so `Base64Decode(input) ?? ""` gives a fallback.
- **`FromJson(text: string)` -> any**. Reads JSON text into a value.
  Returns: The value the text describes: an object, an array, a string, a number, a boolean or null. Null when text is not JSON, holds a number too large to represent, or holds a string with an unpaired surrogate escape; null too when the text is the JSON literal `null`. Text that, read from its start, nests arrays or objects too deeply or passes the host's maximum value size before the point where it stops being JSON is a runtime error, not null.
  Example: `FromJson("{\"a\": [1, 2]}")` returns `{"a": [1, 2]}`; `FromJson("not json")` returns null.
- **`ToJson(value: any)` -> string**. Writes a value as JSON text.
  Returns: The JSON text of value, with no spaces or line breaks between its parts. A number is written in its string form, as `1.5` for `1.50`; null is written as `null`. Properties are written in the order the object holds them. Never null.
  Example: `ToJson({"a": [1, true, null]})` returns `"{\"a\":[1,true,null]}"`.
- **`Sleep(milliseconds: number)` -> null**. Pauses the script for a number of milliseconds.
  Returns: Always null, once the full pause has passed.
  Example: `Sleep(250)` pauses for a quarter of a second.
  Effect: blocking.
- **`Diff(first: object, second: object)` -> object**. Compares two objects property by property.
  Returns: A new object with three properties, each an object: `added` holds each property of second whose name first does not have, with its value in second; `removed` holds each property of first whose name second does not have, with its value in first; `changed` holds, for each name both have with values that are not equal, an object with `from`, the value in first, and `to`, the value in second. Values are compared by value, so equal arrays or objects, and two nulls, are not a change. Only the top-level properties are compared. Never null.
  Example: `Diff({"a": 1, "b": 2}, {"b": 3, "c": 4})` returns `{"added": {"c": 4}, "removed": {"a": 1}, "changed": {"b": {"from": 2, "to": 3}}}`.

### Schema functions

- **`ValidateRequired(source: object, fields: array)` -> object**. Checks that an object has a usable value for each of a list of field names.
  Returns: An object with two properties: `valid`, true when no field failed, and `errors`, an array holding one object of `field` (the name) and `reason` (`"missing"`, `"null"` or `"empty"`) for each name that failed, in the order of fields; a name listed twice that fails is reported twice. Never null.
  Example: `ValidateRequired({ "name": " ", "age": 3 }, ["name", "age", "email"])` returns `{ "valid": false, "errors": [{ "field": "name", "reason": "empty" }, { "field": "email", "reason": "missing" }] }`.
- **`ValidateSchema(data: any, schema: object)` -> array**. Validates a value against a JSON Schema and lists every failure.
  Returns: An array of failures, empty when data is valid. Each failure is an object of `path`, the JSON Pointer of the value that failed (the empty string for data itself, and for `required` and `dependentRequired` the location of the absent property), `keyword`, the keyword that failed, and `message`, the keyword followed by `" at "` and the path, or `"(root)"` for the empty path. `anyOf`, `oneOf`, `not` and `contains` each give one failure at the value, without the failures of their subschemas; `propertyNames` gives one failure at each property whose name fails; `if` is never reported, only the failures of `then` or `else`. The failures are sorted by path and then by keyword, comparing code unit by code unit, and each pair of path and keyword appears once. Never null.
  Example: `ValidateSchema({ "age": "ten" }, { "type": "object", "properties": { "age": { "type": "integer" } }, "required": ["name"] })` returns `[{ "path": "/age", "keyword": "type", "message": "type at /age" }, { "path": "/name", "keyword": "required", "message": "required at /name" }]`.

### Query functions

- **`AnyByField(items: array, fieldName: string, operator: string, value: any)` -> boolean**. Tells whether any object in an array has a field that passes a comparison.
  Returns: True when at least one object's field passes the comparison, false otherwise, and false for an empty array. Never null.
  Example: `AnyByField([{ "n": 1 }, { "n": 7 }], "n", ">", 5)` returns true.
- **`AllByField(items: array, fieldName: string, operator: string, value: any)` -> boolean**. Tells whether every element of an array is an object whose field passes a comparison.
  Returns: True when every element is an object whose field passes the comparison, and true for an empty array; false otherwise. Never null.
  Example: `AllByField([{ "n": 6 }, { "n": 7 }], "n", ">", 5)` returns true.
- **`FindByField(items: array, fieldName: string, operator: string, value: any)` -> any**. Finds the first object in an array whose field passes a comparison.
  Returns: The first object, in array order, whose field passes the comparison: the element itself, not a copy. Null when no object passes, including for an empty array.
  Example: `FindByField([{ "id": 1 }, { "id": 2 }], "id", "==", 2)` returns `{ "id": 2 }`.
- **`WhereByField(items: array, fieldName: string, operator: string, value: any)` -> array**. Selects the objects in an array whose field passes a comparison.
  Returns: A new array of the objects whose field passes the comparison, in their order in items; the objects themselves, not copies. Empty when none passes. Never null.
  Example: `WhereByField([{ "n": 1 }, { "n": 7 }], "n", "<", 5)` returns `[{ "n": 1 }]`.
- **`Select(items: array, fieldName: string)` -> array**. Takes the value of one field from each element of an array.
  Returns: A new array with one element for each element of items, in order: the value of the field, or null when the element is not an object or has no such field. Never null itself.
  Example: `Select([{ "name": "a" }, { "id": 2 }, 5], "name")` returns `["a", null, null]`.
- **`Project(items: array, fields: array)` -> array**. Builds, for each object in an array, a new object with only the listed fields.
  Returns: A new array with one new object for each object in items, in order, holding each listed field in the order of fields, with null for a field the object does not have. Never null.
  Example: `Project([{ "a": 1, "b": 2 }], ["a", "c"])` returns `[{ "a": 1, "c": null }]`.
- **`Omit(items: array, fields: array)` -> array**. Builds, for each object in an array, a new object without the listed fields.
  Returns: A new array with one new object for each object in items, in order, holding every other field of that object in its original order. Never null.
  Example: `Omit([{ "a": 1, "b": 2 }], ["b"])` returns `[{ "a": 1 }]`.
- **`CountIf(items: array, fieldName: string, operator: string, value: any)` -> number**. Counts the objects in an array whose field passes a comparison.
  Returns: The number of objects whose field passes the comparison, a whole number, 0 when none does. Never null.
  Example: `CountIf([{ "n": 1 }, { "n": 7 }, { "n": 9 }], "n", ">=", 7)` returns 2.

### Lambda functions

- **`Map(items: array, callback: function)` -> array**. Builds a new array from the result of a lambda for each element of an array.
  Returns: A new array of the lambda's results, one for each element of items, in the same order; the empty array when items is empty. Never null.
  Example: `Map([1, 2, 3], x => x * 2)` returns `[2, 4, 6]`.
- **`Where(items: array, callback: function)` -> array**. Keeps the elements of an array for which a lambda gives a truthy result.
  Returns: A new array of the elements of items that were kept, in their original order; the empty array when none is kept. Never null.
  Example: `Where([1, 2, 3, 4], x => x > 2)` returns `[3, 4]`.
- **`Reduce(items: array, callback: function, initial: any)` -> any**. Combines the elements of an array into one value, by a lambda applied to each element in turn.
  Returns: The lambda's result for the last element, or initial when items is empty. Null when that value is null: when the last result of the lambda is null, or when items is empty and initial is null.
  Example: `Reduce([1, 2, 3], (total, x) => total + x, 0)` returns 6.
- **`Each(items: array, callback: function)` -> null**. Calls a lambda once for each element of an array.
  Returns: Null, always.
  Example: `Each(items, item => item.count++)` adds one to the `count` of each element of `items`.
- **`Find(items: array, callback: function)` -> any**. Finds the first element of an array for which a lambda gives a truthy result.
  Returns: The first element that matches. Null when no element matches, or when the element that matches is itself null.
  Example: `Find([1, 5, 8], x => x > 4)` returns 5.
- **`All(items: array, callback: function)` -> boolean**. Tells whether a lambda gives a truthy result for every element of an array.
  Returns: True when the lambda's result is truthy for every element, and so true for an empty array; false otherwise. Never null.
  Example: `All([2, 4, 6], x => x % 2 == 0)` returns true.
- **`Any(items: array, callback: function)` -> boolean**. Tells whether a lambda gives a truthy result for at least one element of an array.
  Returns: True when the lambda's result is truthy for at least one element; false otherwise, and so false for an empty array. Never null.
  Example: `Any([1, 3, 4], x => x % 2 == 0)` returns true.
- **`SortBy(items: array, callback: function)` -> array**. Sorts the elements of an array in ascending order of a key that a lambda gives for each.
  Returns: A new array holding the elements of items in ascending order of their keys. Never null.
  Example: `SortBy([{"n": 3}, {"n": 1}], x => x.n)` returns `[{"n": 1}, {"n": 3}]`.
<!-- stdlib-catalog:end -->




## 13. Errors and limits

Compilation errors (parse, validation, linking) stop the script before it runs: unknown names, wrong argument counts, misplaced declarations, a `+` with a string literal, a non-exhaustive `match`. Runtime errors stop the script at the failing statement; `Output` as written so far is returned with the diagnostic. A diagnostic reads `[JM5200] Ln 2, Col 16: Division by zero`: the code, the one-based line and column, and the message.

Common runtime errors and their causes:

| Diagnostic | Cause |
| --- | --- |
| JM5200 `DivisionByZero`, JM5201 `ModuloByZero` | `/` or `%` by zero |
| JM5206 `UnsupportedBinaryOperation` | Arithmetic or `<` on a non-number, or `<` between different types |
| JM5209 `ConcatWithInvalidType` | `&` with an array, object, or function operand |
| JM5210 `StringOperandInAdditionRuntime` | `+` with a string that arrived at run time |
| JM5300 `IndexOutOfRange` | Writing an array element beyond `Length` |
| JM5306 `SetPropertyOnNonObject` | Writing `a.b = v` where `a` is null or not an object |
| JM5100 `InvalidType` | A typed variable or parameter given a value it cannot convert |
| JM5104 `ArgumentTypeMismatch` | A library function given the wrong type |
| JM5105 `CallbackExpected`, JM5106 `CallbackArityMismatch` | A non-lambda, or a lambda with the wrong number of parameters, given to `Map` and the like |
| JM5402 `NotIterable` | `foreach` over a number, boolean, or null value |
| JM5007 `InputValueIsReadOnly` | Writing through a reference into `Input` |
| JM5004 `ScriptFailure` | `fail` |
| JM5900 to JM5903 | The statement, loop, call depth, or time limit |

| Limit | Default |
| --- | --- |
| Execution time | 5 seconds |
| Statements executed | 10,000 |
| Loop iterations, all loops together | 1,000 |
| Call depth | 64 |
| Value size | 16,777,216 elements, properties, or code units |

```jyro
var total = 0
for i in 1 to 5001 do
    total += i
end
```

```text
[JM5901] Ln 2, Col 1: Script execution exceeded maximum loop iteration limit of 1000
```

```jyro
# input: { "n": 5000 }
var values = Range(1, Input.n + 1)
Output.count = Length(values)
Output.total = Sum(values)
Output.evens = Length(Where(values, v => v % 2 == 0))
```

```json
{ "count": 5000, "total": 12502500, "evens": 2500 }
```

## 14. Patterns for business problems

### Filter, transform, aggregate

Covered in section 3. The shape: `Where` to select, `Map` to transform, `Sum`/`Average`/`Length`/`Reduce` to aggregate, `Round` for money.

### Group and summarize

Intermediate results live in variables: a lambda, like a `func`, cannot read `Output`.

```jyro
# input: { "sales": [ { "region": "west", "amount": 100 }, { "region": "east", "amount": 50 }, { "region": "west", "amount": 25 } ] }
var groups = GroupBy(Input.sales, "region")
var totals = {}
foreach region in Keys(groups) do
    totals[region] = Sum(Map(groups[region], s => s.amount))
end
Output.byRegion = totals
Output.regions = Keys(groups)
Output.largest = Last(SortBy(Keys(groups), r => totals[r]))
```

```json
{ "byRegion": { "west": 125, "east": 50 }, "regions": ["west", "east"], "largest": "west" }
```

### Look up one array by another

Build an index object keyed by identifier, then read through it. A missing key gives `null`, and `??` supplies the fallback.

```jyro
# input: { "products": [ { "id": "p1", "name": "Bolt" }, { "id": "p2", "name": "Nut" } ], "orders": [ { "product": "p2", "qty": 3 }, { "product": "p9", "qty": 1 } ] }
var byId = {}
foreach p in Input.products do
    byId[p.id] = p
end
Output.lines = Map(Input.orders, o => {
    product: byId[o.product].name ?? "unknown",
    qty: o.qty
})
Output.unknown = Map(Where(Input.orders, o => not HasProperty(byId, o.product)), o => o.product)
```

```json
{ "lines": [{ "product": "Nut", "qty": 3 }, { "product": "unknown", "qty": 1 }], "unknown": ["p9"] }
```

### Validate and report problems

Collect problems into `Output` when the host expects a report; use `fail` when the host expects a failure.

```jyro
# input: { "order": { "id": 17, "email": "", "items": [] } }
var check = ValidateRequired(Input.order, ["id", "email", "customer"])
var problems = Map(check.errors, e => e.field & " is " & e.reason)
if Length(Input.order.items) == 0 then
    problems = Append(problems, "items is empty")
end
Output.valid = Length(problems) == 0
Output.problems = problems
```

```json
{ "valid": false, "problems": ["email is empty", "customer is missing", "items is empty"] }
```

### Dates and deadlines

```jyro
# input: { "invoiced": "2024-01-15", "paid": "2024-02-03T10:00:00Z", "termsDays": 14 }
var due = DateAdd(Input.invoiced, Input.termsDays, "days")
Output.due = FormatDate(due, "yyyy-MM-dd")
Output.daysLate = Floor(DateDiff(due, Input.paid, "days"))
Output.late = DateDiff(due, Input.paid, "seconds") > 0
Output.month = DatePart(Input.paid, "month")
Output.weekday = DatePart(Input.paid, "dayofweek")
```

```json
{ "due": "2024-01-29", "daysLate": 5, "late": true, "month": 2, "weekday": 6 }
```

### Money

```jyro
# input: { "prices": [19.99, 5.01, 0.1, 0.2] }
Output.total = Sum(Input.prices)
Output.withTax = Round(Output.total * 1.075, 2)
Output.average = Round(Average(Input.prices), 2)
Output.each = Map(Input.prices, p => Round(p * 1.075, 2))
Output.cents = Floor(Output.total * 100)
```

```json
{ "total": 25.3, "withTax": 27.2, "average": 6.33, "each": [21.49, 5.39, 0.11, 0.22], "cents": 2530 }
```

### Build text

```jyro
# input: { "rows": [ { "sku": "A1", "qty": 2 }, { "sku": "B22", "qty": 10 } ] }
var lines = Map(Input.rows, r => PadRight(r.sku, 5) & PadLeft("" & r.qty, 4))
Output.report = Join(lines, "\n")
Output.csv = Join(Map(Input.rows, r => r.sku & "," & r.qty), "\n")
Output.title = ToUpper(Input.rows[0].sku) & " and " & (Length(Input.rows) - 1) & " more"
Output.id = "ORD-" & PadLeft("" & 42, 6, "0")
```

```json
{ "report": "A1      2\nB22    10", "csv": "A1,2\nB22,10", "title": "A1 and 1 more", "id": "ORD-000042" }
```

### Defaults and missing data

```jyro
# input: { "user": { "profile": null, "tags": ["a"], "age": "41" } }
Output.city = Input.user.profile.address.city ?? "unknown"
Output.hasTags = Input.user.tags is array and Length(Input.user.tags) > 0
Output.first = First(Input.user.tags) ?? "none"
Output.known = HasProperty(Input.user, "profile")
Output.count = Length(Input.user.missing)
Output.age = ToNumber(Input.user.age) ?? 0
Output.name = Coalesce([Input.user.nickname, Input.user.name, "anonymous"])
```

```json
{ "city": "unknown", "hasTags": true, "first": "a", "known": true, "count": 0, "age": 41, "name": "anonymous" }
```

### Classify

```jyro
# input: { "scores": [95, 72, 48] }
Output.grades = Map(Input.scores, s => s >= 90 ? "A" : s >= 70 ? "B" : "C")
var labels = []
foreach s in Input.scores do
    switch Floor(s / 10) do
        case 9, 10 then
            labels = Append(labels, "top")
        case 7, 8 then
            labels = Append(labels, "middle")
        default then
            labels = Append(labels, "bottom")
    end
end
Output.labels = labels
```

```json
{ "grades": ["A", "B", "C"], "labels": ["top", "middle", "bottom"] }
```

### Sort, rank, deduplicate

```jyro
# input: { "employees": [ { "name": "Ann", "sales": 300 }, { "name": "Bob", "sales": 500 }, { "name": "Cy", "sales": 400 } ] }
var ranked = SortByField(Input.employees, "sales", "desc")
Output.top2 = Map(Slice(ranked, 0, 2), e => e.name)
Output.best = First(ranked).name
Output.names = Sort(Map(Input.employees, e => e.name))
Output.bands = Distinct(Map(Input.employees, e => Floor(e.sales / 200)))
Output.open = CountIf(Input.employees, "sales", ">=", 400)
Output.anyLow = AnyByField(Input.employees, "sales", "<", 350)
Output.max = Max(Select(Input.employees, "sales"))
```

```json
{ "top2": ["Bob", "Cy"], "best": "Bob", "names": ["Ann", "Bob", "Cy"], "bands": [1, 2], "open": 2, "anyLow": true, "max": 500 }
```

### Nested output

Create each intermediate object or array before writing into it.

```jyro
# input: { "id": 7 }
Output.order = { id: Input.id, status: {} }
Output.order.status.code = "new"
Output.order.history = []
Output.order.history[0] = "created"
Output.order.history = Append(Output.order.history, "queued")
Output.order.count = Length(Output.order.history)
```

```json
{ "order": { "id": 7, "status": { "code": "new" }, "history": ["created", "queued"], "count": 2 } }
```

### Return something other than an object, and stop early

```jyro
# input: { "names": ["b", "a"] }
if Length(Input.names) == 0 then
    Output = []
    exit "nothing to do"
end
Output = Sort(Input.names)
```

```json
["a", "b"]
```

### Iterate an object

```jyro
# input: { "stock": { "bolt": 12, "nut": 0, "washer": 3 } }
var empty = []
foreach name in Keys(Input.stock) do
    if Input.stock[name] == 0 then
        empty = Append(empty, name)
    end
end
Output.empty = empty
Output.totalUnits = Sum(Values(Input.stock))
Output.lines = Map(Keys(Input.stock), k => k & ": " & Input.stock[k])
```

```json
{ "empty": ["nut"], "totalUnits": 15, "lines": ["bolt: 12", "nut: 0", "washer: 3"] }
```

## 15. Checklist before delivering a script

1. Every string join uses `&`; `+` touches numbers only.
2. Every `if`, `while`, `for`, `foreach`, `switch`, `match`, `func`, and `union` closes with `end`; `if` branches use `then`, loops and `switch` use `do`, `elseif` is one word.
3. Comments start with `#`. No semicolons, no braces as block delimiters, no trailing commas.
4. No variable, parameter, or function is named with a reserved word (`end`, `to`, `by`, `in`, `do`, `case`, `default`, `match`, `string`, `number`, `object`, `array`, `function`, ...).
5. Every value added to an array is assigned back: `list = Append(list, x)`. No `.push`, `.length`, or any method call on a value.
6. Lambdas have a single-expression body, declare exactly the parameters the function passes, and are never called directly. Reusable logic is a top-level `func` called from the lambda.
7. `func` declarations are at the top level, read nothing but their parameters and `Input`, and never mention `Output`.
8. `Round` has its decimals argument; `SortByField` has its direction; `Merge` and `Coalesce` receive one array.
9. Intermediate objects exist before nested properties are assigned; `Input` is cloned before it is modified.
10. Loops are bounded by the data's known small size; otherwise the standard library does the iteration.
11. Business-rule violations use `fail "reason"` when the host must see a failure, or are collected into `Output` when the host expects a report.
12. The script writes everything the host needs into `Output` and nothing else; nothing is printed.

## 16. How scripts are run

A host compiles the script once and runs it many times with different `Input`. The command line tool runs one script with a JSON file as `Input`:

```bash
jyro run -s script.jyro -i input.json
```

The tool prints `Output` as JSON. With `-p '{ ... }'` it adds the given JSON as `Input._parameters`. Compilation errors and runtime errors are printed as diagnostics in the form shown in section 13, and the exit code is 1. A host embedding Jyro in .NET supplies `Input` through `JyroRunner.WithInputJson` or `WithInput` and reads `Output` from the result; it may also provide extra functions, which it documents by name and signature, and may lower or raise the limits in section 13.
