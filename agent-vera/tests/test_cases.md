# Vera test cases (dry-run checklist)

Run each command, check the Pass criteria. Use small known PRs/diffs you understand.

## PR mode

| ID | Command | Pass |
|----|---------|------|
| P1 | `/vera pr <valid PR link>` | Max 3 bullets; each has file, function, line numbers, fix; plain language |
| P2 | `/vera pr <link1> <link2>` | One block per PR, given order; closing "fix first" line only if 2+ have HIGH |
| P3 | `/vera pr <PR with no real issues>` | Single line `No blocking issues found.` |
| P4 | `/vera pr <PR with 6+ real issues>` | Exactly 3 bullets + `+N more lower-priority issues not listed.` |
| N1 | `/vera pr` | `Error: vera pr needs at least one PR link.` |
| N2 | `/vera pr https://github.com/o/r` | Invalid PR link error, no work |
| N3 | `/vera pr https://gitlab.com/o/r/pull/1` | Invalid PR link error |
| N4 | `/vera pr <private repo link>` without auth | `Cannot access ...` message, other links still processed |
| E1 | PR link with `/files` or `?diff=split` suffix | Normalised and reviewed |
| E2 | Diff with a hard-coded password | Reported as `[REDACTED]`, secret not printed |

## Diff mode

| ID | Command | Pass |
|----|---------|------|
| D1 | `/vera diff <compare link>` | Exactly 4 sections in order, bullets only |
| D2 | Diff with no SQL/config/AWS changes | Those sections show `- None found.` |
| D3 | Diff adding a new `@Entity` field, no migration | Section 1 flags it with SQL suggestion |
| D4 | Diff reading new `@Value("${x.y}")` key | Section 2 lists key, no value invented |
| D5 | Diff calling `s3.putObject` / `sqs.sendMessage` | Section 4 lists service + permission needed |
| D6 | Diff with a null-pointer risk | Section 3 lists it with file/function/line and fix |
| N5 | `/vera diff` | `Error: vera diff needs exactly one link.` |
| N6 | `/vera diff <link1> <link2>` | Same error |
| N7 | `/vera review <link>` | Unknown mode error |
| E3 | Diff over ~3000 lines | `Partial review: <n> files not read` stated |
| E4 | Any run | Final line `Reviewed: ... Partial: yes|no.` present |

## Quality checks (manual, every release of the skill)

- No invented line numbers (spot-check 3 against the real file).
- No paragraphs in findings.
- Fixes are specific enough to act on without opening the checklist.
- PR output never exceeds 3 bullets; diff output never drops a section.

## Results log

| ID | Date | PASS/FAIL | Notes |
|----|------|-----------|-------|
| | | | |
