# Java as target

Status: unvalidated against a live migration; the points below hold for any recent LTS release.

## Behavior to preserve exactly

- Money and rates: `BigDecimal` with an explicit scale and `RoundingMode` taken from the contract; never `double` or `float`. Build values from strings or integers, compare with `compareTo`.
- Text: decide the charset at every boundary; trailing blank padding, case rules, and collation (use a `Collator` when the legacy order is linguistic, an explicit comparator when it is byte order).
- Dates and times: `java.time` with explicit zones and calendars; map legacy "no date" sentinels to absent values on purpose.
- Null versus empty versus default must follow the rule matrix, not the habit of the language.
- Ordering: always specify it, in queries and in collections.

## Structure

- Write idiomatic Java, organized by business capability; do not mirror programs, indicators, or paragraphs.
- Transactions, isolation, and locking set explicitly and tested; batch jobs are restartable and idempotent, with a recorded checkpoint.
- Keep adapters for the legacy data store behind ports during coexistence, so they can be removed in one batch.
- Every dependency added is justified; build, tests, CI, security scans, and observability follow the target standards.

## Oracle hooks

Expose each migrated unit through the same public interface as the legacy (file, message, API, batch entry) so `characterization-testing` can compare outputs without touching internals.
