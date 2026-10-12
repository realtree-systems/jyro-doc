# Libraries

A library is Jyro source that contains only `func` and `union` declarations. The host attaches libraries to a compilation, and the script calls the library's functions and constructs its variants by name. There is no import or export syntax; a library's declarations are visible to the script as soon as the host attaches it.

```jyro
# fragment
# A library: declarations only.
union Money
    Amount(cents: number, currency: string)
end

func Total(lines)
    return Reduce(lines, (sum, line) => sum + line.qty * line.cents, 0)
end
```

```jyro
# fragment
# A script compiled with the library attached.
Output.total = Amount(Total(Input.lines), "USD")
```

## Contents

Any statement other than a `func` or `union` declaration at the top level of a library is refused with JM3509 `LibraryStatementForbidden`. Function bodies in a library follow the same rules as in a script: they see their parameters, their locals, and `Input`, and cannot reference `Output`.

## Visibility

| From | Can call |
| --- | --- |
| The script | Its own functions and variants, and those of every attached library |
| A library body | Its own functions and variants, those of every other attached library, and the script's variant constructors |

A library body cannot call a function declared in the script. A function name must be unique across the script and every attached library; a script function with a library function's name is refused with JM4103 `FunctionShadowsLibrary`, and variant names are shared in the same way.

## Effect ceiling

The host sets a separate ceiling on the effects a library body may call; by default a library may call functions up to NonDeterministic, which excludes `Sleep`. See [Function Effects](/docs/language/functions/function-effects).

## Artifacts

A compiled program that uses a library cannot be saved as a `.jyrx` artifact. The host API describes how libraries are attached, parsed once, and reused; see [Libraries](/docs/host-api/compiling/libraries) in the Host API section.
