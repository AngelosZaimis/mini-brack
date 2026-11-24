---
name: review-lens
description: Review the current branch (or a given path) through one learning lens such as SOLID, modern Java or transactions. Reports what is applied well, what is worth applying, and what would be overkill.
argument-hint: <topic> [path]
disable-model-invocation: true
allowed-tools: Bash(git diff:*) Bash(git merge-base:*) Bash(git ls-files:*) Bash(git rev-parse:*) Read Grep Glob
---

# Review lens

A learning review, not a bug hunt. The user writes the code; this skill points at places and asks questions.
Never write or edit code here, and never give a finished fix.

Arguments: `$ARGUMENTS` → first word is the topic, optional second word is a path.

## 1. Resolve the topic

- Topics are the files in `references/` next to this file (`references/<topic>.md`).
- No topic, or an unknown one: list the available topics and stop.

## 2. Resolve the target

**Path given:** review every file under it (respecting the topic's Scope). No diff involved.

**No path:** the whole branch against `main`.
1. `base=$(git merge-base main HEAD)`
2. Changed tracked files (committed on the branch + uncommitted): `git diff --name-only $base`
3. New untracked files: `git ls-files --others --exclude-standard`
4. Drop anything under `build/`, `.gradle/`, `.idea/` and binaries, then filter by the topic's Scope.
5. Nothing left: say so, suggest passing a path, and stop. Do not fall back to the whole module.

Read each target file in full for context. For tracked files, get the changed lines with `git diff $base -- <file>`;
untracked files count as fully changed. **Findings are only about changed lines.** At most one note may mention
nearby unchanged code, marked "(existing code)".

## 3. Load context

- The topic file: its Scope and Checks.
- The ADRs named in the topic's Scope, plus the titles of all ADRs in `docs/adr/` (needed for the breach check).

## 4. Review

For each check in the topic file, look at the changed code:
- Matches **Look for** and not **Skip when** → candidate for 🔧 Worth applying.
- Matches **Look for** but **Skip when** holds, or the textbook move would be overkill → candidate for 🚫 Don't apply here.
- The code already does what the check asks, in a non-trivial way → candidate for ✅ Applied well.
- The textbook move contradicts an accepted ADR → ⚖️ Trade-off vs ADR, never 🔧.

Independently of the topic, scan the changed code for breaches of any accepted ADR (e.g. editing an applied
migration, changing `ddl-auto`, non-UUID ids, version numbers in module build files, layer packages).

Rank candidates by learning value, not severity.

## 5. Report

Use exactly this shape. Omit empty groups except 🔧 (write "nothing this time" if empty).

```
## <Topic> lens: <branch or path> (<n> files)

❗ ADR breaches            (all of them, only if any)
- file:line: ADR 000X. <what breaks>.

✅ Applied well            (max 2)
- file:line: <CHECK>. <why it is good, one line>.

🔧 Worth applying          (max 3)
- file:line: <CHECK>. <what you see, one line>.
  Ask yourself: <the check's question, adapted to this code>

🚫 Don't apply here        (max 2)
- file:line: <CHECK>. <textbook move> → <why it is overkill here>.

⚖️ Trade-off vs ADR        (max 1)
- <lens suggests X>; ADR 000X chose Y. Revisit if: <condition>.

+<n> more in <group>, ask. Pick one to discuss?
```

Rules:
- Every finding has `file:line` and the check name it comes from.
- 🔧 gives a question or a hint, never code.
- Keep each line short. No intro, no summary paragraph.
- End with the "+n more" line (only if something was cut) and "Pick one to discuss?".
