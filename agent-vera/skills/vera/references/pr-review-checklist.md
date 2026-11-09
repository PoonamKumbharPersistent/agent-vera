# PR review checklist (QA)

Use as a thinking aid. Do not output this list.
Output shape is defined in `SKILL.md` Mode 1: What/why (1–2 lines), top 3 findings, optional review-comment tally, then four release-prep sections (SQL, properties, bugs, AWS).

## Re-review

- Always load PR review comments/reviews when present.
- Prefer still-open feedback that the current diff has not fixed.
- Do not restate issues clearly fixed in the new diff unless a regression remains.

## Priority rubric

- **HIGH** — will break QA, corrupt data, leak data/secrets, or fail under normal load.
- **MED** — works but is fragile, hard to test, or will hurt at scale.
- **LOW** — maintainability; only list if nothing higher exists.

## 1. Correctness

- Wrong conditions, off-by-one, inverted checks, missing `else` branches.
- Null / empty / missing-value handling on every new input.
- Changed behaviour for existing callers (signature, return type, defaults).
- Date/time zone and rounding mistakes; money handled as floats.
- Copy-paste logic that was not updated.

## 2. Error handling and resilience

- Swallowed exceptions (`catch` with no log or rethrow).
- No timeout, retry or fallback on network/DB/queue calls.
- Retried operations that are not idempotent (double charge, duplicate record).
- Partial failure leaves data half-written (missing transaction / rollback).

## 3. Security

- Missing authentication/authorisation checks on new endpoints.
- User input in SQL, shell commands, file paths, or logs without validation.
- Secrets, tokens, passwords or PII in code, config, or log lines.
- Overly broad CORS/permissions; sensitive data returned in API responses.

## 4. Scale and performance (industry practice)

- Query inside a loop (N+1); `SELECT *`; missing index for a new filter or join.
- Unbounded result sets: no pagination, limit, or streaming.
- Loading large collections/files fully in memory.
- Blocking calls on hot paths; missing caching for repeated reads.
- Shared mutable state / race conditions under concurrent requests.
- New batch/cron job that can overlap with itself.
- Lock scope too wide; long transactions.
- Will this still work with 10x data and 100x traffic?

## 5. Tests

- New logic with no test; tests that assert nothing.
- Only the happy path covered; no negative or boundary cases.
- Tests that depend on real external services or clock.

## 6. API and data contracts

- Breaking changes to request/response shape without versioning.
- Database changes without a migration, or a migration that cannot be rolled back.
- Hard-coded environment values (URLs, IDs, bucket names).

## 7. Release-prep (always assess; output as the four sections in SKILL.md)

Use the same criteria as `gitdiff-checklist.md`, scoped to this PR’s diff only:

- **SQL:** new/changed tables, columns, indexes, functions, native queries, missing migrations.
- **Properties:** new/renamed keys, env vars, feature flags, secret/parameter lookups.
- **Bugs from new code:** nulls, wrong conditions, races, transactions, N+1, swallowed errors, broken callers (max 5 in section 3).
- **AWS:** S3, SQS/SNS, Lambda, IAM, Secrets Manager, Parameter Store, OTEL collectors, SG/VPC, schedules — only if the diff implies them.

## 8. Maintainability (only when it causes real risk)

- Function doing too many things, duplicated blocks, dead code.
- Misleading names, magic numbers, unexplained complex logic.

## Plain-language phrasing examples

- Instead of "N+1 query anti-pattern": "This loads one DB row at a time inside a loop; with 1,000 users it makes 1,000 DB calls. Fix: fetch all in one query."
- Instead of "non-idempotent retry": "If this call is retried, the customer can be charged twice. Fix: send a unique request ID and skip if already processed."
