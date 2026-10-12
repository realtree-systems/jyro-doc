# Function Bodies

## Scope

A function body starts with an empty scope. It sees:

- its parameters;
- the variables it declares;
- `Input`;
- the functions and variant constructors it can call by name.

A variable declared outside the function is not visible inside it, and `Output` cannot be referenced in a body. Values a function needs are passed as arguments, and its result is returned with `return`.

```jyro
var rate = 0.2
func Tax(amount)
    return amount * rate
end
```

```text
[JM3100] Ln 3, Col 21: Undeclared variable 'rate'
```

```jyro
func Tax(amount, rate)
    return amount * rate
end
Output.tax = Tax(100, 0.2)
```

```json
{ "tax": 20 }
```

## Return value

The value of a call is the value given to `return`, or `null` when the body ends without a `return` or with a bare `return`. See [Return](/docs/language/statements/return).

## Arguments

Arguments are passed by sharing. A function receives a reference to an array or an object, and a change to its contents is visible to the caller. See [Reference Semantics](/docs/language/types-and-values/reference-semantics).

## Recursion and call depth

A function may call itself, directly or through other functions. Each active call occupies one level of call depth, standard library and host functions included. A call that exceeds the depth limit set by the host ends the run with JM5902 `CallDepthLimitExceeded`. The default limit is given in [Resource Limits](/docs/language/errors-and-limits/resource-limits).

```jyro
func Down(n)
    return Down(n + 1)
end
Output.a = Down(0)
```

```text
[JM5902] Ln 2, Col 12: Script execution exceeded maximum call depth limit of 64
```

## Exit and fail

`exit` and `fail` inside a function end the whole run, not just the call. See [Exit](/docs/language/statements/exit) and [Fail](/docs/language/statements/fail).

## Lambdas in functions

A function may take lambdas as arguments and return a lambda. A lambda written in a function body captures the function's parameters and local variables.

```jyro
func Above(limit)
    return n => n > limit
end
Output.a = Where([1, 5, 10], Above(4))
```

```json
{ "a": [5, 10] }
```
