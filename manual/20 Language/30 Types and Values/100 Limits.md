# Limits

The following limits on values and source are fixed. They cannot be changed by the host. The limits on execution time, statement count, loop iterations, and call depth are set by the host; see [Resource Limits](/docs/language/errors-and-limits/resource-limits).

| Limit | Value | Diagnostic |
| --- | --- | --- |
| Value size: the code units of a string, the elements of an array, or the properties of an object | 16,777,216 (2^24) | JM5919 `ValueSizeLimitExceeded` |
| Value nesting depth, for operations that walk a value | 128 | JM5910 `ValueNestingLimitExceeded` |
| Source nesting depth, for expressions, statements, and literals | 256 | JM2002 `SourceNestingLimitExceeded` |
| Generated length, for a string or array built by `PadLeft`, `PadRight`, `RandomString`, or `Range` | 1,000,000 | JM5702 `GeneratedLengthExceeded` |

The value size limit applies to every value a script makes with an operator, an assignment, or a standard library function, and to the input the host supplies. The first value of `Output`, when the host supplies one, is exempt.

The value nesting limit applies to equality, to `Clone`, to conversion to and from JSON, and to freezing `Input`. A value that is nested more deeply can exist but cannot be walked.
