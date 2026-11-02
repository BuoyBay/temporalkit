# TemporalKit 0.1.0

Import `temporalkit` from a sealed package. Requires existing dateparsekit and
timezonekit 0.1.0 packages. No host locale or native calendar parser is used.

Public API contracts:

* `New TemporalDateOnly(year, month, day)` validates Gregorian years 1..9999.
  Read-only Year/Month/Day; `ToInvariant()` is yyyy-MM-dd.
* `New TemporalTimeOfDay(hour, minute, second = 0, millisecond = 0)` validates
  0..23, 0..59, 0..59, 0..999. Read-only fields; invariant HH:mm:ss.fff.
* `New TemporalDurationMillis(milliseconds)` stores every signed 64-bit value.
  Read-only Milliseconds; invariant `[-]H+:mm:ss.fff`, hours exceed 24.
  Zero formats unsigned; parsing rejects plus signs and negative zero.
* `AnalyzeTemporalDate(text, pattern = "yyyy-MM-dd")`,
  `AnalyzeTemporalTime(text, pattern = "HH:mm:ss.fff")`, and
  `AnalyzeTemporalDuration(text)` return `TemporalEntry`: State is exactly
  Empty/Incomplete/Valid/Invalid; ErrorCode and zero-based Position identify
  the failure (positions count Unicode codepoints; Position is consumed length for Empty/Incomplete/Valid).
  Pattern errors, cross-field calendar errors and final duration overflow use
  position zero; duration digit accumulation overflow identifies the digit.
  DateValue/TimeValue/DurationValue are Nil except for the matching Valid result.
  Strict parse callers accept only Valid. No trimming or inferred date order.
* `RequireTemporalDate(text, pattern = "yyyy-MM-dd")`,
  `RequireTemporalTime(text, pattern = "HH:mm:ss.fff")`, and
  `RequireTemporalDuration(text)` return the corresponding value or throw
  InvalidArgumentException for Empty/Incomplete/Invalid, with state/code/position.
* `FormatTemporalDate(value, pattern)` / `FormatTemporalTime(value, pattern)`
  are display formatters; `ToInvariant` remains independent of display patterns.
  Invalid values/patterns and precision loss throw InvalidArgumentException.
* `FormatTemporalTimeDisplay(value, pattern)` explicitly permits display-only
  truncation of omitted seconds/fractions. It leaves the immutable value unchanged.
  Use the strict formatter and a full-precision pattern for edit text.
* `ResolveTemporalLocal(date, time, ianaName, overlap)` returns ZonedDateTime;
  overlap must be exactly reject/earlier/later (case sensitive). Gaps always reject.
* `FormatTemporalInstant(instant, ianaName, datePattern, timePattern)` uses
  timezonekit fields, returning date + one ASCII space + time. This is display
  formatting: a shorter time pattern may omit seconds/fractions without changing
  the instant. Strict time editing still requires the precision-preserving API.
* `TemporalInstantFromTimestamp(text)` delegates strict offset timestamp parsing
  to dateparsekit and returns DateTime. Its existing precision rules apply.

Pattern grammar (ASCII, 1..128 characters): date contains exactly one each of
`yyyy`, `MM`, `dd` in any order; time contains exactly `HH` or `hh`, `mm`,
optionally `ss`, optionally `f`/`ff`/`fff` after seconds, and exactly one `tt`
for `hh` (uppercase AM/PM). Tokens may be reordered. Each neighboring token
must have a nonempty literal between them. Unquoted literals are ASCII space,
`-`, `/`, `:`, `.`, `,`; other printable ASCII literals must be single quoted.
A literal apostrophe is expressed as `''` inside a quoted literal. No two-digit
years, month names, native long styles, timezone pattern tokens, optional fields,
variable widths or localized digits. Literal text matches exactly. Fractions
represent decimal seconds; omitted fields must be zero when formatting, and
fewer than three fractional digits require exact divisibility (no truncation).

Entry analysis rejects numeric prefixes that cannot reach a valid field range.
A prefix is Incomplete while the consumed fields are valid and a token or literal
is unfinished. Cross-field Gregorian validity is checked once all date fields
are present. An Incomplete result is an editing state, not a parsed value.
An invalid pattern yields Invalid/Pattern even for empty input. Input is bounded
at 256 characters. Error codes: Pattern, Length, Literal, Digit, Range,
Overflow, NonCanonical; successful states have an empty code.
String APIs cannot recover NUL bytes discarded by an earlier byte-to-String
conversion; callers must validate wire bytes before decoding untrusted data.

Named zones retain timezonekit's pinned IANA data and UTC coverage
1800-01-01 inclusive through 2501-01-01 exclusive. Out-of-range local resolution,
unknown zones and invalid overlap policy throw InvalidArgumentException. Elapsed
durations are independent of calendar dates and daylight saving transitions;
there is no duration-to-calendar arithmetic API. Values are immutable references.
