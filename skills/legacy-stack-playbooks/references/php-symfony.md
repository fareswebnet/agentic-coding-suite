# PHP and Symfony (legacy to migrate, or version to uplift)

Status: unvalidated against a live migration. Console command names exist in recent Symfony versions; check the installed version, and note that some commands appear only from a given release.

## Inventory sources

- Dependencies and versions: `composer.json` and `composer.lock` (Symfony version, abandoned or unmaintained packages, forked bundles), the PHP version, and the extensions required.
- Entry points: the router (`bin/console debug:router --show-controllers`, remembering routes can come from attributes, annotations on versions before 7, YAML, XML, PHP configuration, or imported bundles), console commands (`bin/console list`), message handlers and transports (`debug:messenger`), scheduled work (the Symfony scheduler on versions that have it, stable from 6.4, run by a messenger worker; plus cron, systemd timers, and supervisor files outside the repository, including worker flags such as limits that hide restarts), webhooks, and event listeners and subscribers (`debug:event-dispatcher`, per event such as `kernel.request` to see the order). Also useful: `debug:validator`, `debug:form`, `debug:translation`, `lint:container`, `lint:twig`, `messenger:failed:show`, and `doctrine:migrations:status`.
- Wiring: the service container (`debug:container`, `debug:autowiring`), configuration per environment (`debug:config`, `.env` files, secrets), Twig templates and extensions (`debug:twig`), form types, validator constraints, serializer groups and normalizers.
- Data: Doctrine mapping (`doctrine:mapping:info`, `doctrine:schema:validate`), migrations history, SQL filters, lifecycle callbacks and listeners, raw SQL in repositories.
- Security: firewalls and authenticators (`debug:firewall`), the `access_control` rules read from `security.yaml` or `debug:config security` (first matching rule wins), the role hierarchy, voters, user providers, remember-me, and exposed debug tooling.
- Usage: web server and application logs, APM, Monolog channels, and shell, cron, supervisor, or CI logs for console commands (Symfony keeps no execution history of its own); probe in production read-only with approval.
- Static quality: PHPStan or Psalm baselines, deprecation logs, and Rector rules show how far the code is from the next version.

## Hidden behavior to turn into rules or questions

- Business rules inside Doctrine lifecycle callbacks and listeners, event subscribers (and their priority), form data transformers, validation groups, and request listeners that change locale or tenant.
- Formatting and rules in Twig filters and extensions (money, dates, pluralization).
- PHP typing: loose comparison and numeric strings, integer overflow to float, `round()` behavior (it changed in PHP 8.4, so state the PHP version when comparing results), PHP 8 changes to numeric-string comparison, arrays used as both lists and ordered maps, `DateTime` modifiers such as "last day of next month", float versus `bcmath` or decimal libraries for money.
- Dates: default time zone, Doctrine date types without a zone, daylight saving boundaries, and locale or `intl` formatting.
- Null versus empty string coming from forms (`empty_data`, trimming), and request parameters always arriving as strings.
- Doctrine: flush ordering, cascades, orphan removal, soft-delete filters, transaction boundaries, optimistic locking, second-level cache.
- Messenger: retries, failure transport, ordering and idempotency of handlers; work started in a request but finished asynchronously.
- State outside the code: sessions, HTTP cache, application cache, feature flags, and environment-dependent parameters.
- Security: `access_control` order and first-match behavior, role hierarchy, voters, CSRF settings, mass assignment through forms; record them in the security baseline.

## Test-oracle design

Record real responses and database effects of the running application (HTTP requests, console commands, message handlers) and replay them against the target. Freeze the clock, time zone, and locale, use the same fixtures, and compare response bodies, status codes, headers that carry meaning, rows written, messages emitted, files, and mails. Cover form validation errors, permission denials per role, and the paths owned by listeners rather than controllers.

## Strategy hints

- A Symfony version upgrade is an uplift: follow the supported upgrade path version by version, fix deprecations first, and keep the characterization suite green at each step.
- When moving away from Symfony, port capabilities (rules, contracts, jobs), not controllers and bundles one by one.
