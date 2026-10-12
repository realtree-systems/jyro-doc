# Function Effects

Every function has an effect, which describes what a call to it may do beyond computing a result.

| Effect | Meaning |
| --- | --- |
| Pure | The result depends only on the arguments. |
| NonDeterministic | The function reads the clock or a random source, so two calls with the same arguments may give different results. |
| Blocking | The function may pause execution. |
| Mutating | The function reads or writes state outside the script. |

The effects are ordered as listed. A standard library function or a host function declares its effect. The effect of a function declared with `func` is the highest effect of any function its body calls; a body that calls only pure functions is pure.

In the standard library, `Now`, `Today`, `NewGuid`, `RandomInt`, `RandomChoice`, and `RandomString` are non-deterministic, `Sleep` is blocking, and every other function is pure.

## Effect ceilings

The host sets the highest effect a script may call, and separately the highest effect a library body may call. A call to a function whose effect is above the ceiling is refused when the script is compiled, with JM4104 `FunctionEffectExceedsMax`. The ceilings are set with `WithMaxEffect` and `WithLibraryMaxEffect` on the compiler; see [Effects](/docs/host-api/host-functions/effects).

By default a script may call functions of every effect, and a library body may call functions up to NonDeterministic, which excludes `Sleep`.
