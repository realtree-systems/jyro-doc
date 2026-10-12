# Conditional Operator

The expression `condition ? a : b` gives `a` when `condition` is truthy and `b` otherwise. Only the chosen branch is evaluated.

```jyro
# input: { "score": 72 }
Output.grade = Input.score >= 50 ? "pass" : "fail"
Output.safe = true ? 1 : 1 / 0
```

```json
{ "grade": "pass", "safe": 1 }
```

The operator is right-associative. A conditional in the `b` position needs no parentheses.

```jyro
# input: { "n": 0 }
var n = Input.n
Output.sign = n > 0 ? "positive" : n < 0 ? "negative" : "zero"
```

```json
{ "sign": "zero" }
```

The condition is read at the `??` level and the branches are full expressions. A conditional in the condition position or in the `a` position requires parentheses. See [Operator Precedence](/docs/language/expressions/operator-precedence).
