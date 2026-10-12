# Statement Forms

A script is a sequence of statements. Statements are written one after another with no separator; a line end is not required between them. A statement is one of the following.

| Statement | Form |
| --- | --- |
| Declaration | `var name [: type] [= expression]` |
| Assignment | `target [: type] = expression`, or `target op= expression` |
| If | `if condition then … [elseif condition then …] [else …] end` |
| Switch | `switch expression do case … [default then …] end` |
| Match | `match expression do case … end` |
| While | `while condition do … end` |
| For | `for name in start to end [by step] do … end`, or `downto` |
| Foreach | `foreach name in expression do … end` |
| Break, continue | `break`, `continue` |
| Return | `return [expression]` |
| Exit | `exit [expression]` |
| Fail | `fail [expression]` |
| Delete | `delete target` |
| Function declaration | `func Name(parameters) … end` |
| Union declaration | `union Name variants end` |
| Expression | Any expression; its value is discarded |

## Blocks

The body of a compound statement is a block: a sequence of statements that ends at the keyword `end`, or at the next `elseif`, `else`, `case`, or `default` of the same statement. There are no braces. Each block is a scope; see [Scope](/docs/language/variables/scope).

```jyro
var n = 3
if n > 2 then var big = true Output.big = big end
```

```json
{ "big": true }
```

## Expression statements

An expression on its own is a statement. It is evaluated and its value is discarded. A call made for its side effects on an argument, and an increment or decrement, are the usual cases.

```jyro
var list = [1]
var count = 0
count++
Length(list)
Output.count = count
```

```json
{ "count": 1 }
```

## Statement count

Every executed statement, including `break` and `continue`, counts toward the run's statement limit. See [Resource Limits](/docs/language/errors-and-limits/resource-limits).
