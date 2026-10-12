# Null Coalescing

The expression `a ?? b` gives `a` when `a` is not `null`, and `b` otherwise. Only `null` is replaced: `false`, `0`, and `""` are kept.

```jyro
# input: { "name": null, "count": 0 }
Output.a = Input.name ?? "anonymous"
Output.b = Input.count ?? 10
Output.c = Input.missing ?? "default"
Output.d = false ?? true
```

```json
{ "a": "anonymous", "b": 0, "c": "default", "d": false }
```

The right operand is evaluated only when the left operand is `null`.

```jyro
Output.a = 1 ?? 1 / 0
```

```json
{ "a": 1 }
```

`??` is right-associative, so a chain gives the first operand that is not `null`.

```jyro
Output.a = null ?? null ?? 3
```

```json
{ "a": 3 }
```

`??` binds less tightly than `or` and more tightly than the conditional operator: `0 ?? 1 or 2` is read as `0 ?? (1 or 2)`. See [Operator Precedence](/docs/language/expressions/operator-precedence).
