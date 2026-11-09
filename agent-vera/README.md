# Vera

A human-sounding code reviewer with two modes. Vera is short for "verify".

| Mode | Command | Env | Output |
|------|---------|-----|--------|
| PR review | `/vera pr <PR link> [<PR link> ...]` | QA | Max 3 priority bullets per PR: file/function, lines, plain-language fix |
| GitDiff review | `/vera diff <diff link>` | PROD | 4 bullet sections: new SQL, config changes, bugs introduced, AWS config |

## Folder layout

```text
agent-vera/
├── README.md                          <- this guide
├── skills/vera/
│   ├── SKILL.md                       <- the agent (instructions + output formats)
│   └── references/
│       ├── pr-review-checklist.md     <- QA review rubric (best practices + scale)
│       └── gitdiff-checklist.md       <- PROD release checklist (SQL/config/bugs/AWS)
├── tests/test_cases.md                <- dry-run test cases
└── output/                            <- optional saved reviews (--save)
```

## Step-by-step: build and install

### Step 1. Decide the contract (done for you)

Write down, before any code: inputs, outputs, limits, errors. Vera's contract is in `SKILL.md`: 3 bullets max for PR, 4 fixed sections for diff, bullets only, no invented line numbers.

### Step 2. Create the skill folder

A skill is a folder with a `SKILL.md` that starts with YAML front matter:

```yaml
---
name: vera
description: <what it does + when to use it + the commands>
---
```

The `description` is how the harness decides to load the skill, so include the trigger words (`PR review`, `git diff`, `/vera`).

### Step 3. Write the instructions in `SKILL.md`

Keep this order (same pattern as Sage):

1. Commands table
2. Global rules (tone, evidence, no secrets)
3. How to fetch the code
4. Link validation + exact error messages
5. One section per mode with the exact output template
6. Hard rules (the invariants repeated at the end)

### Step 4. Put big knowledge in `references/`

Keep `SKILL.md` short. Put checklists in `references/*.md` and tell the skill which one to read per mode. This keeps token use low and makes the checklists easy to edit without touching the logic.

### Step 5. Give it access to the code

The skill can only review what it can fetch. Choose one:

- **GitHub CLI (recommended):** use the absolute binary `C:\Users\PoonamKumbhar\personal\tools\gh_2.103.3_windows_amd64\bin\gh.exe` (not bare `gh`, not a relative path). Run `"C:\Users\PoonamKumbhar\personal\tools\gh_2.103.3_windows_amd64\bin\gh.exe" auth login`. Works for private repos.
- **HTTP fetch:** append `.diff` to PR/compare/commit URLs. Public repos only unless you pass a token.
- **Cloned repo:** if the repo is local, the skill can read full files for context.

Without access, Vera replies "Cannot access <link>" instead of guessing.

### Step 6. Install

Point the harness at `skills/vera/` (the folder that contains `SKILL.md`). Link that folder; do not copy the markdown files. Restart the session, then run `/vera pr <link>` or `/vera diff <link>`.

### Step 7. Test

Run `tests/test_cases.md` against 3 or 4 PRs you know well. Check that line numbers are real, PR output never exceeds 3 bullets, and every diff section appears.

### Step 8. Improve from real runs

When Vera misses or over-reports, add the lesson to a checklist file (not the main rule list), re-run the failed test, and log it in the results table.

## Other things you need for a good skill

1. **Clear trigger description** so it loads at the right time.
2. **Strict output templates** so results are identical every run and easy to scan.
3. **Evidence rule** ("report only what the diff shows") to prevent made-up findings.
4. **Severity rubric** (HIGH/MED/LOW definitions) so "top 3" is consistent.
5. **Tests with pass and fail criteria**, including error paths.
6. **Access and auth plan** (`C:\Users\PoonamKumbhar\personal\tools\gh_2.103.3_windows_amd64\bin\gh.exe` login, tokens, private repos, rate limits).
7. **Large-input policy** (partial review, review riskiest files first, say so).
8. **Secret handling** (never echo keys or passwords from a diff).
9. **Read-only guardrails** (never push, comment or modify repos unless you add a separate mode for it).
10. **Team context file (optional):** a `references/team-standards.md` with your stack (Java/Spring, Node, DB type, AWS accounts, naming rules, migration tool such as Flyway or Liquibase). Vera gets far sharper with this.
11. **Optional later upgrades:**
    - Post the review as a PR comment through `"C:\Users\PoonamKumbhar\personal\tools\gh_2.103.3_windows_amd64\bin\gh.exe" pr comment` (add as a third, opt-in mode).
    - Run on a schedule or in CI when a PR opens.
    - Add a `--save` log and a weekly summary of recurring issues.
    - Split into sub-agents (security, performance, SQL) if you want an "agent army" style setup.

## Quick usage

```text
/vera pr https://github.com/org/repo/pull/101 https://github.com/org/repo/pull/102
/vera diff https://github.com/org/repo/compare/release-1.4...release-1.5
/vera diff https://github.com/org/repo/commit/<sha> --save
```
