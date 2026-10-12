# For

```jyro
# fragment
for name in start to end [by step] do
    statements
end

for name in start downto end [by step] do
    statements
end
```

A `for` loop counts from `start` toward `end`. With `to` the values ascend and with `downto` they descend. The `end` value is exclusive: the loop runs while the variable is less than `end` for `to`, and greater than `end` for `downto`. The default step is `1`.

```jyro
var up = []
for i in 0 to 5 do
    up = Append(up, i)
end
var down = []
for i in 5 downto 0 do
    down = Append(down, i)
end
var stepped = []
for i in 0 to 10 by 3 do
    stepped = Append(stepped, i)
end
Output.up = up
Output.down = down
Output.stepped = stepped
```

```json
{ "up": [0, 1, 2, 3, 4], "down": [5, 4, 3, 2, 1], "stepped": [0, 3, 6, 9] }
```

`start`, `end`, and `step` are evaluated once, in that order, before the first iteration. `start` and `end` must be numbers; they need not be integers. `step` must be an integer of at least one. The value in iteration *k*, counting from zero, is `start + k * step` for `to` and `start - k * step` for `downto`.

```jyro
var values = []
for i in 1.5 to 4 do
    values = Append(values, i)
end
for i in 3 to 3 do
    values = Append(values, "never")
end
Output.values = values
```

```json
{ "values": [1.5, 2.5, 3.5] }
```

```jyro
for i in 0 to 10 by 0 do
end
```

```text
[JM5907] Ln 1, Col 1: for requires an integer step of at least 1. Received: 0
```

```jyro
for i in "a" to 10 do
end
```

```text
[JM5206] Ln 1, Col 1: Unsupported binary operation < for string and number
```

## The loop variable

The loop variable is declared by the loop and is visible only in the body. It is a fresh variable in each iteration. Assigning to it changes the value for the rest of that iteration only; the next iteration takes the next value of the sequence.

```jyro
var seen = []
for i in 0 to 3 do
    i = 100
    seen = Append(seen, i)
end
Output.seen = seen
```

```json
{ "seen": [100, 100, 100] }
```

`break` ends the loop and `continue` advances to the next value; see [Break and Continue](/docs/language/statements/break-and-continue). Each iteration counts toward the run's loop iteration limit.
