# Java as target

Status: unvalidated against a live migration; the points below target the recent long-term-support releases (17, 21, 25).

## Behavior to preserve exactly

- Money and rates: `BigDecimal` with an explicit scale and `RoundingMode` taken from the contract; never `double` or `float`. Build values from strings or integers (never `new BigDecimal(double)`), compare with `compareTo`, and pass a `MathContext` or scale to `divide`, which throws on non-terminating results. `DecimalFormat` rounds half-even by default.
- Text: decide the charset at every boundary; trailing blank padding, case rules, and collation (use a `Collator` when the legacy order is linguistic; for true byte order compare the encoded bytes as unsigned values, since `String.compareTo` compares UTF-16 code units and matches neither UTF-8 nor EBCDIC order). Set the default charset explicitly on runtimes older than 18, and avoid locale-dependent `String.format`, `toUpperCase`, and `toLowerCase` without an explicit locale.
- Dates and times: `java.time` with explicit zones and calendars; map legacy "no date" sentinels to absent values on purpose.
- Null versus empty versus default must follow the rule matrix, not the habit of the language.
- Ordering: always specify it, in queries and in collections; hash-based collections have no stable iteration order.
- Arithmetic: integer overflow is silent, use the exact-arithmetic methods where it matters; pin the time-zone data version when dates cross historical rule changes.

## Structure

- Write idiomatic Java, organized by business capability; do not mirror programs, indicators, or paragraphs.
- Transactions, isolation, and locking set explicitly and tested; batch jobs are restartable and idempotent, with a recorded checkpoint.
- Keep adapters for the legacy data store behind ports during coexistence, so they can be removed in one batch.
- Every dependency added is justified; build, tests, CI, security scans, and observability follow the target standards.

## Oracle hooks

Expose each migrated unit through the same public interface as the legacy (file, message, API, batch entry) so `characterization-testing` can compare outputs without touching internals.
