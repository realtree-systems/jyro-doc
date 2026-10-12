# Lambdas

A lambda is an expression that evaluates to a function value. It has a parameter list, the arrow `=>`, and a body that is a single expression.

```jyro
# fragment
x => expression
(x) => expression
(a, b) => expression
() => expression
```

A lambda with one parameter may omit the parentheses. The body extends as far to the right as possible.

A lambda is passed to a standard library function or a host function that calls it. A lambda cannot be called from script code: a call must be made on a function name, and a variable holding a lambda is not one.

```jyro
# input: { "orders": [ { "id": 1, "total": 50 }, { "id": 2, "total": 150 } ] }
Output.large = Where(Input.orders, o => o.total > 100)
Output.ids = Map(Input.orders, o => o.id)
Output.sum = Reduce(Input.orders, (acc, o) => acc + o.total, 0)
```

```json
{ "large": [{ "id": 2, "total": 150 }], "ids": [1, 2], "sum": 200 }
```

A lambda may be stored in a variable or in a container and passed later.

```jyro
var isEven = n => n % 2 == 0
Output.evens = Where([1, 2, 3, 4], isEven)
```

```json
{ "evens": [2, 4] }
```

## Captured variables

A lambda body sees the variables in scope where the lambda is written, and `Input`. A captured variable is read when the lambda runs, so a later change to the variable is visible to the lambda. The variable of a `for` or `foreach` loop is a fresh variable in each iteration, so a lambda made in a loop keeps that iteration's value.

```jyro
var threshold = 1
var above = n => n > threshold
threshold = 2
Output.a = Where([1, 2, 3], above)
```

```json
{ "a": [3] }
```

A lambda body cannot reference `Output`.

## Arity

A function that takes a lambda calls it with a fixed number of arguments, and the lambda must declare exactly that many parameters. The check is made when the lambda is called.

```jyro
Output.a = Map([1], (a, b) => a)
```

```text
[JM5106] Ln 1, Col 12: Map calls its callback with 1 argument(s), but the callback declares 2
```

A value that is not a function, including `null`, is refused with JM5105 `CallbackExpected`.

```jyro
Output.a = Map([1], null)
```

```text
[JM5105] Ln 1, Col 12: Map expected a lambda callback but got null
```

## Function names

A function declared with `func`, a standard library function, and a host function are not values and cannot be passed. A lambda that calls the function is passed instead.

```jyro
Output.a = Map([1], Length)
```

```text
[JM3100] Ln 1, Col 21: Undeclared variable 'Length'
```

```jyro
func Twice(n)
    return n * 2
end
Output.a = Map([1, 2], x => Twice(x))
```

```json
{ "a": [2, 4] }
```

## Identity

Each evaluation of a lambda expression makes a new function value. Two function values are equal only when they are the same value.

```jyro
var f = x => x
Output.a = f == f
Output.b = (x => x) == (x => x)
```

```json
{ "a": true, "b": false }
```
