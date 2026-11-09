# GitDiff review checklist (PROD)

Use as a thinking aid. Output follows the four-section format in `SKILL.md`.

## 1. New SQL needed

Look for in the diff:

- New/changed entity or model classes (`@Entity`, `@Column`, ORM models, DTO-to-table mappings) with no matching migration file.
- New `@Query`, native SQL, stored-procedure calls, query-builder code referencing tables/columns/indexes.
- Changed column type, length, nullability or default.
- New foreign keys, unique constraints, enums stored as strings/ints.
- New reference/seed data the code assumes exists (statuses, roles, lookup rows, feature flags in DB).
- New filters/joins/sorts on large tables that need an index.
- Data back-fill needed for existing rows when a new NOT NULL column is added.

Flag:

- Migration is not backward compatible (old app version running during deploy would break): suggest expand-then-contract.
- Destructive change (drop/rename column or table): needs a rollback plan.
- Long-running `ALTER` on a large table: suggest off-peak or online schema change.

## 2. Property / config changes needed

Look for:

- New keys read via `@Value`, `@ConfigurationProperties`, `process.env`, `os.getenv`, `System.getenv`, `config.get`.
- Changes in `application*.properties|yml`, `.env*`, Helm values, Docker/compose env, `appsettings*.json`.
- Renamed or removed keys that still exist in PROD config.
- Feature flags and toggles (default value for PROD).
- Environment-specific values: hostnames, URLs, bucket/queue names, ARNs, timeouts, thread-pool sizes, credentials references.
- Secrets that should come from Secrets Manager / Parameter Store, not a property file.

Output the key name and what kind of value goes in; never print real secret values.

## 3. Bugs this change can cause

Look for:

- Null/empty handling on new paths; unchecked `Optional.get()`, missing `.map` on null.
- Wrong or inverted conditions; missing `else`; off-by-one.
- Behaviour changes to existing callers or API responses.
- Missing `@Transactional`, wrong transaction boundary, partial writes.
- Race conditions, shared mutable state, non-thread-safe collections.
- N+1 queries, unbounded queries, loading big lists in memory.
- Swallowed exceptions, missing timeouts/retries, retries on non-idempotent calls.
- Resource leaks (unclosed streams, connections, threads).
- Hard-coded values that only work in QA/dev.
- Time zone, locale, rounding, encoding issues.

Report max 5, highest priority first.

## 4. AWS configuration needed

Look for new or changed use of:

- **S3:** bucket, prefix, presigned URLs, lifecycle, CORS, encryption.
- **SQS/SNS/EventBridge:** queue/topic names, DLQ, visibility timeout, subscriptions, schedules.
- **Lambda/ECS/EKS/EC2:** new function/task, env vars, memory/timeout, scaling, image tags.
- **RDS/DynamoDB/ElastiCache/OpenSearch:** new table, index, parameter group, capacity.
- **IAM:** new API calls in code (`s3:PutObject`, `sqs:SendMessage`, `secretsmanager:GetSecretValue`, `kms:Decrypt`...) need matching role permissions.
- **Secrets Manager / Parameter Store / KMS:** new secret/parameter names.
- **Networking:** new outbound hosts, security group/port, VPC endpoint.
- **Observability:** new log groups, metrics, alarms for a new job.
- **API Gateway / ALB:** new routes, timeouts, auth.

State the exact resource and the exact permission or setting needed, with the file and line that implies it.

## PROD-specific reminders

- Anything environment-specific must have a PROD value ready before deploy.
- Prefer changes that can be rolled back without data loss.
- If the diff is partial or large, say so; do not claim "None found" for files not read.

## Optional branch sync (`--branch`)

When the user passes a branch name with `/vera diff`:

- Compare `develop...<branch>` per distinct repo from the links.
- Anything still on the branch and not in `develop` is a gap: prefix `MISSING in develop:`.
- If the compare has no ahead commits/files → `All changes from <branch> are present in develop.`
- Commits differ but file tree empty → content is present; note commits only.
