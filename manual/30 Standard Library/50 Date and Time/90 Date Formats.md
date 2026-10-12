<!-- hand-written -->
# Date Formats

Dates and times are strings. `ParseDate`, `FormatDate`, `DateAdd`, `DateDiff`, and `DatePart` read a date with the same rules, and `Now`, `ParseDate`, and `DateAdd` write one with the same rules.

## Reading a date

A date is read in the form of RFC 3339: a full date, optionally followed by `T` and a time, optionally followed by a fraction of a second and a time offset.

```text
YYYY-MM-DD
YYYY-MM-DDTHH:mm:ss
YYYY-MM-DDTHH:mm:ss.fff
YYYY-MM-DDTHH:mm:ssZ
YYYY-MM-DDTHH:mm:ss.fff+HH:mm
```

| Rule | Detail |
| --- | --- |
| Date alone | Midnight, UTC |
| Time with no offset | UTC |
| Offset | `Z`, `z`, or `+HH:mm` / `-HH:mm`; the instant is converted to UTC |
| `T` | `T` or `t`; a space is refused |
| Seconds | Required when a time is given; a second of `60` is read as `59` |
| Fraction | Any number of digits; kept to the millisecond |
| Day | Must exist in its month; year `0000` is a leap year |
| Range | `0001-01-01T00:00:00.000Z` to `9999-12-31T23:59:59.999Z`, after the offset is applied |

A time without seconds, an offset without a colon, text before or after the date, and any other form are refused. `ParseDate` returns `null` for a string it cannot read; the other functions raise JM5802 `DateFormatInvalid`.

```jyro
Output.a = ParseDate("2024-01-15")
Output.b = ParseDate("2024-01-15T10:30:00+02:00")
Output.c = ParseDate("2024-01-15t10:30:00.1234z")
Output.d = ParseDate("15/01/2024")
```

```json
{
  "a": "2024-01-15T00:00:00.000Z",
  "b": "2024-01-15T08:30:00.000Z",
  "c": "2024-01-15T10:30:00.123Z",
  "d": null
}
```

## Writing a date

A date is written as `yyyy-MM-ddTHH:mm:ss.fffZ`, always in UTC with three fraction digits. `Today` writes `yyyy-MM-dd`. There is no local time and no time zone name.

## Format strings

`FormatDate` takes a format string made of the following tokens. Any other letter, or a run of a letter of a length not listed, is refused with JM5801 `DateFormatStringInvalid`. Characters other than letters and quotes are copied to the output.

| Token | Output |
| --- | --- |
| `yyyy` | The year, four digits |
| `MM`, `M` | The month, two digits or without a leading zero |
| `dd`, `d` | The day of the month, two digits or without a leading zero |
| `HH`, `H` | The hour, 00 to 23, two digits or without a leading zero |
| `mm`, `m` | The minute, two digits or without a leading zero |
| `ss`, `s` | The second, two digits or without a leading zero |
| `fff` | The millisecond, three digits |
| `T`, `Z` | The letter itself, any number of times |
| `'text'` | The text inside the quotes; `''` inside the quotes is one quote |

```jyro
var date = "2024-01-05T09:05:07.250Z"
Output.a = FormatDate(date, "dd/MM/yyyy HH:mm")
Output.b = FormatDate(date, "d M yyyy")
Output.c = FormatDate(date, "yyyy-MM-ddTHH:mm:ss.fffZ")
Output.d = FormatDate(date, "'Day' d 'of the year'")
```

```json
{
  "a": "05/01/2024 09:05",
  "b": "5 1 2024",
  "c": "2024-01-05T09:05:07.250Z",
  "d": "Day 5 of the year"
}
```

## Units and parts

`DateAdd` and `DateDiff` take a unit name, in the singular or the plural, compared without regard to ASCII case: `day`, `week`, `month`, `year`, `hour`, `minute`, and `second`. Any other name is refused with JM5804 `DateUnitInvalid`. `DateDiff` gives a fractional count for `day`, `week`, `hour`, `minute`, and `second`, and a whole calendar count for `month` and `year`.

`DatePart` takes a part name: `year`, `month`, `day`, `hour`, `minute`, `second`, `dayofweek`, or `dayofyear`. `dayofweek` is `0` for Sunday through `6` for Saturday. Any other name is refused with JM5805 `DatePartInvalid`.

```jyro
Output.a = DateAdd("2024-01-31", 1, "month")
Output.b = DateDiff("2024-01-01", "2024-01-02T12:00:00Z", "days")
Output.c = DateDiff("2024-01-31", "2024-03-01", "months")
Output.d = DatePart("2024-01-15", "dayofweek")
Output.e = DatePart("2024-12-31", "dayofyear")
```

```json
{ "a": "2024-02-29T00:00:00.000Z", "b": 1.5, "c": 1, "d": 1, "e": 366 }
```

A result outside the range of dates is refused with JM5806 `DateOutOfRange`.
