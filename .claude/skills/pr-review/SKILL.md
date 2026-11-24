---
name: pr-review
description: Pre-merge review of the current branch or a PR against MiniBrack's rules. Finds bugs, security issues, ADR/CLAUDE.md breaches, missing tests. Read-only; optionally posts one summary comment on the PR.
argument-hint: [pr-number] [--post]
disable-model-invocation: true
allowed-tools: Bash(git diff:*) Bash(git merge-base:*) Bash(git ls-files:*) Bash(git rev-parse:*) Bash(git log:*) Bash(git fetch:*) Bash(git show:*) Bash(gh pr view:*) Bash(gh pr diff:*) Read Grep Glob
---

# PR review

A gate before merging, not a lesson (that is `review-lens`). Find what should block or improve the merge.
Never write or edit code. Never post anything unless `--post` was given.

Arguments: `$ARGUMENTS` → optional PR number, optional `--post`.

## 1. Target

**No PR number:** the current branch against `main`.
1. `base=$(git merge-base main HEAD)`
2. Changed files: `git diff --name-only $base` plus untracked `git ls-files --others --exclude-standard`

**PR number `n`:**
1. `gh pr view n --json title,body,headRefName,baseRefName,files,author`
2. `git fetch origin pull/n/head` then `base=$(git merge-base origin/main FETCH_HEAD)`
3. Changed files: `git diff --name-only $base FETCH_HEAD`; read them with `git show FETCH_HEAD:<path>`

Drop `build/`, `.gradle/`, `.idea/`, binaries. Nothing left: say so and stop.
Read each changed file in full; findings are only about changed lines.

## 2. Context

- `CLAUDE.md` and every accepted ADR in `docs/adr/`.
- For workflow files: other files in `.github/` so pins and permissions can be compared.

## 3. Checks

| Check | Look for |
|---|---|
| Bug | wrong logic, null/empty cases, off-by-one, broken equals/hashCode, wrong transaction boundary, resource leaks |
| Security | injection (SQL, JPQL, shell, `${{ }}` in workflow `run:`), secrets or credentials in code/config, over-broad workflow `permissions`, unpinned actions, `pull_request_target`, sensitive data in logs |
| ADR / CLAUDE.md | edited applied migration, `ddl-auto` not `validate`, non-UUID-v7 id or DB default on id, layer packages, version numbers in module build files, repositories outside `settings.gradle.kts`, env-specific values in base `application.yml`, postgres image out of sync |
| Tests | changed logic without a test; integration test not using `TestcontainersConfiguration` |
| Comments | more comments than a human would write, comments that restate the code, AI-style explanation blocks |
| Attribution | `Co-authored-by: Claude`, "Generated with Claude" or similar in commits, PR body or files (`git log $base..HEAD --format=%B`) |

Skip formatting (spotless does it) and style taste. No finding is better than a weak one.

## 4. Report

```
## PR review: <branch or #n> (<n> files)

🔴 Must fix
- file:line: <Check>. <problem>. Fix: <hint, no code>.

🟡 Should fix
- file:line: <Check>. <problem>. Fix: <hint>.

🔵 Consider          (max 3)
- file:line: <Check>. <one line>.

Verdict: Ready to merge | Fix first
```

- Omit empty groups. No findings: "Nothing found." plus the verdict.
- One line per finding, `file:line` always. No intro, no praise, no summary paragraph.
- 🔴 for bugs, security holes, ADR breaches. Any 🔴 → "Fix first".

## 5. Post (only with `--post` and a PR number)

Post the report as one comment: `gh pr comment n --body-file <tmp file>`. This asks for permission; that is intended.
Plain report only, no signature or tool footer.
