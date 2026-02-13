---
name: code-simplifier
description: Simplify and refine existing code for clarity, consistency, and maintainability while preserving exact behavior. Use when asked to refactor, clean up, reduce complexity, improve readability, or make code more idiomatic—especially for code in a git diff (working tree, staged/index), specific commit(s), or a recent change set.
---

# Code Simplifier

## Workflow (diff-first, behavior-preserving)

0. Confirm constraints and decision points (default: conservative)
   - Default goals: preserve behavior, keep diff small, keep edits within the chosen scope
   - If there are multiple reasonable refactor paths, present options + tradeoffs and let the user decide before applying non-trivial changes
   - If any decision items exist, do a review-first pass (produce the formatted output) and wait for the user’s choice before implementing those items

1. Establish scope (default: recently modified code)
   - Working tree diff: `git diff`
   - Staged/index diff: `git diff --cached`
   - A specific commit: `git show <sha>` (or `git show <sha> --name-only`)
   - A commit range (change set): `git diff <base>..<sha>` (or between two SHAs)
   - A branch/PR-style diff: `git diff <base-branch>...HEAD` (triple-dot)
   - If the user points to files/directories, use that scope instead
   - Derive the file list from the chosen scope (`--name-only`) and keep edits within it unless expanding scope is required to preserve behavior
   - File-list examples:
     - `git diff --name-only`
     - `git diff --cached --name-only`
     - `git show --name-only --pretty='' <sha>`
     - `git diff --name-only <base>..<sha>`
     - `git diff --name-only <base-branch>...HEAD`

2. Understand behavior before refactoring
   - Identify inputs/outputs, side effects, error modes, and any invariants
   - Follow project conventions from `AGENTS.md`, linters/formatters, and surrounding code
   - If the repo has a style guide (e.g. `CONTRIBUTING.md`, `STYLEGUIDE.md`, `CLAUDE.md`), apply it
   - Expand context *only as needed* to preserve behavior:
     - Scan call sites/usages of changed exports/symbols (`rg`), plus adjacent modules affected by the change
     - Check API boundaries (public exports, CLI flags, HTTP handlers, DB queries, serialization) for contract stability
     - If a function/type signature changed, check all direct/indirect dependents and update them explicitly (do not rely on “it compiles” alone)

3. Apply safe simplifications (preserve functionality)
   - Reduce nesting with guard clauses/early returns
   - Consolidate duplicated logic and remove dead/unused code
   - Improve names for clarity; keep naming consistent with the codebase
   - Prefer explicit, readable code over dense one-liners
   - Avoid nested ternary operators; prefer `if/else` chains or `switch`
   - Keep refactors local; extract helpers only when it improves understanding (avoid “abstraction for its own sake”)
   - Prefer small, mechanical transformations; preserve evaluation order and side-effect ordering
   - Separate two categories of work:
     - Safe mechanical cleanups: apply directly (no decision needed)
     - Opinionated/structural refactors: propose as options first

4. Avoid refactors that change behavior unless explicitly requested
   - Do not change public APIs, wire formats, query semantics, or error contracts
   - Do not introduce new dependencies or reformat unrelated code
   - Do not “optimize” by making code clever or less debuggable
   - Do not remove useful abstractions or merge unrelated concerns “for fewer lines”

5. Verify
   - Run the narrowest relevant checks first (targeted tests), then broader checks if available (typecheck/lint/build)
   - If no tests exist, prefer minimal, local changes and add tests only when the project already has a test harness
   - If a formatter is standard in the repo, run it on touched files only (avoid large, unrelated diffs)
   - Do not stage, commit, or rewrite history unless the user explicitly asks

6. Communicate changes (code-review style, with coaching)
   - List touched files + the scope used (working tree / staged / commit / range)
   - Summarize meaningful simplifications (what got simpler and why)
   - Call out any *behavior-risk* areas you checked (callers/contracts) and why they’re safe
   - Add short training notes when helpful (recurring patterns, preferred conventions), without lecturing
   - If you expanded scope beyond the diff/commit range, state exactly what and why

## Quick checklist

- Preserve outputs, side effects, and error behavior
- Preserve ordering (evaluation order, iteration order, and side-effect timing)
- Watch for language-specific semantic traps (e.g. JS/TS: `||` vs `??`, `==` vs `===`, async/concurrency changes)
- Keep performance characteristics broadly similar unless the refactor is demonstrably equivalent
- Remove redundant comments; keep comments that explain intent or non-obvious constraints
- Keep changes scoped to the diff/commit range; expand only to affected dependencies

## Output format (code review style, decision-friendly)

Goal: mimic a PR review (GitHub/GitLab-style): short summary + categorized comments + explicit decisions.
Keep it one-screen when possible; avoid repeating information across sections.
Write content in the user’s language; keep headings stable.

Severity guide:
- `P0` likely bug/contract break
- `P1` risky/fragile behavior
- `P2` maintainability/readability
- `P3` style/nit

Rules:
- Prefer ≤3 decisions and ≤10 total comments
- If any decisions exist, stop after the review report and ask the user to reply with picks (e.g. `D1=A, D2=B`)
- Only add `## Appendix (details)` if the user asks; include key `rg` queries, caller/contract checks performed, and any non-obvious semantic pitfalls

## Review summary

- Scope: `staged` / `working tree` / `commit` / `range`
- Files: short list (use “+N more” if long)
- Status: `REQUEST_CHANGES` (any `P0/P1`) / `COMMENT` (only `P2/P3`) / `APPROVE` (no issues)
- Assumptions: 1–3 bullets on behavior/contracts/style guides

## Decisions (only if needed)

- D1: question
  - A (Recommended): smallest diff that improves clarity
  - B: deeper cleanup with larger diff
  - Pick: `A` or `B`

## Requested changes (P0/P1)

- [P?] `file:symbol` — Problem: … Impact: … Fix: … (Decision: `D1` if applicable)

## Suggestions (P2)

- [P2] `file:symbol` — Problem: … Impact: … Fix: …

## Nits (P3)

- [P3] `file:symbol` — Nit: … Fix: …

## Next steps

- After picks (if any): list the concrete edits you will apply (group by file, keep it short)
- Verify: 1–3 narrow commands (tests/lint/typecheck/build)
- Coaching: 1–2 actionable habits (omit if none)
