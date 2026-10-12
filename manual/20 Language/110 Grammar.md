# Grammar

The grammar is given in EBNF. `{ x }` is zero or more, `[ x ]` is optional, and `|` separates alternatives. Terminal symbols are quoted. White space and comments may appear between any two tokens; the places where a line end is significant are described in [Source Text](/docs/language/lexical-structure/source-text).

## Statements

```text
script          = { statement } ;
block           = { statement } ;

statement       = func_decl | union_decl | var_decl
                | if_stmt | while_stmt | for_stmt | foreach_stmt
                | switch_stmt | match_stmt
                | return_stmt | exit_stmt | fail_stmt
                | break_stmt | continue_stmt | delete_stmt
                | assignment | expression ;

var_decl        = "var" identifier [ ":" type ] [ "=" expression ] ;
assignment      = expression [ ":" type ] assign_op expression ;
assign_op       = "=" | "+=" | "-=" | "*=" | "/=" | "%=" | "&=" ;

if_stmt         = "if" expression "then" block
                  { "elseif" expression "then" block }
                  [ "else" block ] "end" ;
while_stmt      = "while" expression "do" block "end" ;
for_stmt        = "for" identifier "in" expression ( "to" | "downto" ) expression
                  [ "by" expression ] "do" block "end" ;
foreach_stmt    = "foreach" identifier "in" expression "do" block "end" ;

switch_stmt     = "switch" expression "do"
                  { "case" expression { "," expression } "then" block }
                  [ "default" "then" block ] "end" ;
match_stmt      = "match" expression "do" match_case { match_case } "end" ;
match_case      = "case" identifier [ bindings ] "then" block ;
bindings        = "(" [ identifier { "," identifier } ] ")" ;

return_stmt     = "return" [ expression ] ;
exit_stmt       = "exit" [ expression ] ;
fail_stmt       = "fail" [ expression ] ;
break_stmt      = "break" ;
continue_stmt   = "continue" ;
delete_stmt     = "delete" expression ;

func_decl       = "func" identifier "(" [ parameter { "," parameter } ] ")" block "end" ;
parameter       = identifier [ ":" type ] [ "=" expression ] ;

union_decl      = "union" identifier variant { variant } "end" ;
variant         = identifier [ "(" [ field { "," field } ] ")" ] ;
field           = identifier [ ":" type ] ;

type            = "any" | "number" | "string" | "boolean" | "object"
                | "array" | "null" | "function" ;
```

The target of an assignment, an increment, or a `delete` must be an identifier, a property access, or an index access; the grammar accepts any expression and validation refuses the others. The expression after `return`, `exit`, or `fail` must begin on the same line as the keyword. A parameter default must be a literal or `-` before a number literal.

## Expressions

```text
expression      = conditional ;
conditional     = coalesce [ "?" expression ":" expression ] ;
coalesce        = or_expr [ "??" coalesce ] ;
or_expr         = and_expr { "or" and_expr } ;
and_expr        = not_expr { "and" not_expr } ;
not_expr        = "not" not_expr | type_check ;
type_check      = equality [ "is" [ "not" ] type ] ;
equality        = relational { ( "==" | "!=" ) relational } ;
relational      = concatenation { ( "<" | "<=" | ">" | ">=" ) concatenation } ;
concatenation   = additive { "&" additive } ;
additive        = multiplicative { ( "+" | "-" ) multiplicative } ;
multiplicative  = unary { ( "*" | "/" | "%" ) unary } ;
unary           = "++" postfix | "--" postfix | "-" unary | postfix ;
postfix         = primary { call_args | "." any_identifier | "[" expression "]" | "++" | "--" } ;

call_args       = "(" [ named_args | positional_args ] ")" ;
positional_args = expression { "," expression } ;
named_args      = named_arg { "," named_arg } ;
named_arg       = any_identifier ":" expression ;

primary         = lambda | "null" | "true" | "false" | number | string
                | array_literal | object_literal | match_expr
                | "(" expression ")" | identifier ;

lambda          = lambda_params "=>" expression ;
lambda_params   = identifier | "(" [ identifier { "," identifier } ] ")" ;

array_literal   = "[" [ expression { "," expression } ] "]" ;
object_literal  = "{" [ property { "," property } ] "}" ;
property        = ( any_identifier | string ) ":" expression ;

match_expr      = "match" expression "do" match_expr_case { match_expr_case } "end" ;
match_expr_case = "case" any_identifier [ bindings ] "then" expression ;
```

`call_args` is accepted only when the expression before it is a single identifier, and `call_args`, `"[" expression "]"`, and the postfix `"++"` and `"--"` are accepted only on the same line as the expression before them.

## Lexical elements

```text
identifier      = identifier_start { identifier_char } ;    (not a keyword)
any_identifier  = identifier_start { identifier_char } ;    (a keyword is accepted)
identifier_start = "_" | letter ;
identifier_char = "_" | letter | decimal_digit ;

number          = decimal | hex | binary ;
decimal         = ( "0" | nonzero_digit { digit } ) [ "." digit { digit } ]
                  [ ( "e" | "E" ) [ "+" | "-" ] digit { digit } ] ;
hex             = "0" ( "x" | "X" ) hex_digit { hex_digit } ;
binary          = "0" ( "b" | "B" ) ( "0" | "1" ) { "0" | "1" } ;

string          = '"' { string_char | escape } '"'
                | "'" { string_char | escape } "'" ;
escape          = "\" ( "n" | "r" | "t" | "\" | '"' | "'" | "0"
                | "u" hex_digit hex_digit hex_digit hex_digit ) ;

comment         = "#" { any character except a line end } ;
```

`letter` is any character of Unicode general category L and `decimal_digit` any character of category Nd. `string_char` is any character other than the closing quote and `\`; a line end is permitted. The keywords are listed in [Keywords](/docs/language/lexical-structure/keywords).
