# Scope

Variables are block scoped. The body of every `if`, `elseif`, `else`, `while`, `for`, `foreach`, `case`, `default`, and `match` case, and the body of every function, is a block with its own scope. A variable is visible from its declaration to the end of the block that contains it.

```jyro
var outer = 1
if true then
    var inner = 2
    outer = outer + inner
end
Output.outer = outer
```

```json
{ "outer": 3 }
```

A variable is not visible before its declaration, and a declaration is not hoisted.

```jyro
Output.a = missing
```

```text
[JM3100] Ln 1, Col 12: Undeclared variable 'missing'
```

## Redeclaration and shadowing

A name cannot be declared twice in the same scope. A declaration in an inner block may reuse the name of a variable in an outer block; the inner variable hides the outer one until the inner block ends. The initializer of the inner declaration is evaluated before the new variable comes into scope, so it can read the outer variable.

```jyro
var x = 1
if true then
    var x = x + 10
    Output.inner = x
end
Output.outer = x
```

```json
{ "inner": 11, "outer": 1 }
```

```jyro
var n = 1
var n = 2
```

```text
[JM3101] Ln 2, Col 1: Variable 'n' is already declared
```

## Loop variables

The variable of a `for` or `foreach` loop is scoped to the loop body and is a fresh variable in each iteration. A lambda created in one iteration captures that iteration's value. The variable is not visible after the loop, and the body cannot declare another variable with its name.

## Function bodies

A function body starts with an empty scope. It sees its parameters, the variables it declares, `Input`, and the functions and variant constructors it can call by name. A variable declared outside the function is not visible inside it, and the function cannot reference `Output`. See [Function Bodies](/docs/language/functions/function-bodies).

```jyro
var factor = 2
func Scale(n)
    return n * factor
end
```

```text
[JM3100] Ln 3, Col 16: Undeclared variable 'factor'
```

A parameter counts as declared in the outermost block of the body, so the body cannot declare a variable with a parameter's name at that level.

## Lambdas

A lambda body sees the variables that are in scope where the lambda is written, and reads their current values when it runs. See [Lambdas](/docs/language/expressions/lambdas).
