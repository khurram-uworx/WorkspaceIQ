---
name: iterative-work
description: Build mode only — do not use in plan mode. Use when executing discrete steps of multi-step implementation work. Suggests/asks for a feature branch and switches to it, writes the plan to docs/TODO.md first, grounds the plan via microsoft-learn MCP and raises any decisions/red flags to the human (Gate G1), commits locally after each logical change unit (does NOT push), verifies each change per AGENTS.md Testing Guidance, files deferred work as a GitHub issue via gh the moment it is found (delegated to a subagent, after checking for an existing open issue), runs a two-gate review (branch as a whole + against docs/TODO.md) before deleting docs/TODO.md (Gate G2) and deletes only when both clear, then offers to push and create/update a PR. Push and PRs are always human-confirmed. Escalate decisions to the human rather than assume; prefer proper engineering over quick wins.
disable-model-invocation: true
---

# Iterative Work Workflow

When working on multi-step tasks, commit after each logical change unit so the human can review incremental progress.

Work as proper engineering, not quick wins: rigor (grounding, reviews, escalation) beats speed, and a
decision or red flag is never resolved by assumption when the human can be asked.

## Rules

1. **Commit frequently** — after completing each discrete step (bug fix, feature addition, refactor, test update, etc.)
2. **Never push** — commits are local only. The human pushes manually after reviewing changes outside the session.
3. **Write clear commit messages** — describe what changed and why, in imperative mood.
4. **Stage selectively** — only stage files relevant to the completed step, not unrelated changes.
5. **Verify before committing** — run the smallest meaningful check for the layer you changed before each commit (see AGENTS.md → Testing Guidance: backend `.\mvnw.cmd test` in `src/backend`, frontend `npm test` / `npm run build` in `src/frontend`, Playwright from `tests/PlaywrightTests` for import/login/cross-stack flows, `dotnet test` only for the WiqsServer side under `tests/Wiqs.Tests`). If a check cannot be run, say exactly which checks were skipped and why.
6. **One logical change per commit** — don't bundle unrelated changes together.
7. **File deferred work the moment you find it** — if a step surfaces a limitation, follow-up, or refactor that is outside the current plan, open a GitHub issue for it immediately (see "Follow-ups are GitHub issues"). Don't rely on memory or wait until the plan finishes — compaction can lose it.
8. **Involve the human at decision points** — if a plan, grounding pass, review, or failure surfaces a decision or red flag, stop and ask rather than assume — especially when the blast radius is more than trivial. Quick wins are a warning sign, not the goal.

## Plan-first workflow

Persist the plan before executing so it is saved at highest fidelity, even if context is later lost.

1. **Suggest/ask for the branch** — propose a short feature branch name (e.g., `khurram/<feature>`), ask the human to confirm, then create it off the current base (typically `main`): `git checkout -b <branch>`. Do not proceed until the human confirms the branch.
2. **Write the plan to `docs/TODO.md` first** — document the problem, proposed changes (with code sketches where useful), verification steps, planned commit list, and a note that deferred work discovered during execution is filed as a GitHub issue the moment it is found — don't rely on memory or wait until the end of the plan, as compaction during execution can lose important items (see "Follow-ups are GitHub issues"). Commit it as its own logical unit (`docs: plan <work> in TODO.md`).
3. **Ground the plan (Gate G1)** — immediately after the plan commit, and before any implementation, ground the plan via the microsoft-learn MCP server and codebase navigation (code-memory MCP), and state the blast radius (see "Grounding the plan"). If grounding surfaces any decision to make, any red flag, or any assumption that needs resolving, **stop and involve the human** — never resolve it by assumption. Implementation begins only after the human clears the plan (as written or amended).
4. **Execute iteratively** — complete one logical change at a time, committing after each (see Workflow below).
5. **Review and conclude (Gate G2)** — before deleting `docs/TODO.md`, run the two pre-deletion reviews — (a) the branch work as a whole, and (b) the branch work against `docs/TODO.md` (see "Branch review (from a distance)"). Only when both clear, remove the plan and commit the removal (`git rm docs/TODO.md` → `docs: remove TODO.md — plan executed`). If a review surfaces an issue, fix it as an additional commit and re-run both reviews; if a decision is needed with more than trivial blast radius, involve the human. Leave the plan in place only if an item is still pending.
6. **Offer push + PR** — report the completed work, then offer to push the branch and create (or update) a pull request on the remote. Ask explicitly; do not push or open a PR without the human's confirmation. Push remains human-controlled by default.

## Quality gates (proper engineering, not quick wins)

Two hard gates check the default fast path. Their whole point is to stop the work at the two moments
where an agent is most tempted to skip rigor: before implementation (planning) and before declaring
done (plan deletion).

- **G1 — Grounding gate (right after the plan is committed).** Ground the plan in official
  documentation and examples via the microsoft-learn MCP server, navigate the codebase with
  code-memory MCP, and state the blast radius. If grounding surfaces any decision to make, any red
  flag, or any assumption baked into the plan, **stop and involve the human** — never resolve it by
  assumption. Implementation begins only after the human clears the plan (as written or amended).
- **G2 — Pre-deletion review gate (right before `docs/TODO.md` is deleted).** Run two reviews of the
  work that just landed: (a) the branch work as a whole, and (b) the branch work against
  `docs/TODO.md` item by item. Only when both clear may the plan be deleted and the work concluded.
  If a review surfaces an issue, fix it as an additional commit and re-run both reviews. If a
  decision is needed with more than trivial blast radius, involve the human — do not take the
  assumption yourself.

## Branch review (from a distance)

This is the work that clears **G2** — it runs on the branch, **before** `docs/TODO.md` is deleted and
therefore before anything is pushed. The branch is the cheapest place to catch problems: the diff is
compact, one logical change per commit, and nothing has hit `main`. It is a fresh, critical pass —
not a re-run of step-by-step work.

### (a) The branch work as a whole

1. **Re-read the plan's problem and acceptance criteria** — for each problem and acceptance criterion recorded in `docs/TODO.md` (and any ticket the human referenced), compare it against the corresponding commit. Ask per item:
   - Is the root cause actually fixed (not just the symptom)?
   - Are **all** acceptance criteria met, literally?
   - Is the fix minimal and scoped, with no unrelated changes bundled in?
2. **Inspect each commit's diff** — `git show <hash>` / `git diff main...<branch> --stat`. Look for:
   - Edits that are dead code (defined but never called) or leave similar dead code behind.
   - Silent failure modes the fix claims to eliminate but doesn't fully cover (e.g. an Excel import path that accepts a header but still mis-maps a column, a UI path that swallows an API error instead of showing it).
   - Correctness under the edge cases the plan claimed to handle (flexible/shuffled/extra Excel columns, CRLF-wrapped headers, leading blank rows, empty uploads, error responses) — not just the happy path.
3. **Verify the tests pin the acceptance criteria** — do the added tests actually assert the failure mode from the plan (e.g. the exact header-matching or parsing case), or just a happy path? Are edge cases (missing or duplicate headers, case/whitespace differences, chunked imports) covered?

### (b) The branch work against `docs/TODO.md`

Read the plan top to bottom and map every item to landed, verified work:

1. **Problem** — does each stated problem correspond to what the branch actually addresses?
2. **Proposed changes** — does the landed diff match the plan's shape (parser, validation, DTOs, migrations) with no silent drift?
3. **Verification steps** — was each planned verification run, and did it pass (build, targeted tests, full suite)?
4. **Planned commits** — does the branch log contain every planned commit, plus only additive fix/test commits?
5. **Blast radius** — does the documented blast radius still match reality after implementation?
6. **Follow-ups** — every deferred item captured during execution has a GitHub issue (not lost), and each is still accurate.

### Completing the gate

- **Fix before deleting** — if either review surfaces a real gap, fix it properly (regression test + update the affected docs if the behavior changed) as an **additional commit** on the same branch, then re-run both reviews. If the blast radius is more than trivial, confirm the direction with the human first.
- **Escalate decisions** — never resolve a review finding by assumption. If the right call is not unambiguous, stop and ask the human.
- **Dispose of residual concerns** — anything real but out of scope → open a GitHub issue for it immediately via a subagent; never hold it in memory.
- **Conclude** — only once both reviews clear does the workflow delete `docs/TODO.md` and move on to "Offer push + PR".

## Follow-ups are GitHub issues

Deferred work is tracked as **GitHub issues** on `khurram-uworx/WorktreeIQ`, not in a
local markdown file. The tracker exists and `gh` is already authenticated, so a
concern becomes real only once the issue is open.

**Delegate issue creation to a subagent.** Listing and reading existing issues,
drafting the body, and creating the issue produce a lot of output that has no
bearing on the implementation work in progress. Keep that out of the main
context:

```
subagent(agent="general", description="File follow-up issue", prompt="<below>")
```

Give the subagent:

- A precise one-line title for the concern.
- The full detail: what was observed, file paths, symptom, why it is out of
  scope for the current plan, and what the next person would need to know.
- The commit hash or branch where it was found.
- Instruction to report back **only** the issue number and URL.

The subagent should then:

1. **Check for an existing issue first — this is required.** Run
   `gh issue list --state all --limit 100` and `gh issue list --state open
   --search "<keywords>"`. If an open issue already covers it, add a comment with
   the new context instead of creating a duplicate, and report that issue's
   number. Only create a new issue when nothing covers it.
2. Check `gh label list` and apply an existing label (`bug`, `enhancement`,
   `documentation`, ...) rather than inventing one.
3. Write the body to a file and use `gh issue create --body-file <path>`;
   multi-line `--body` strings are easy to mangle in PowerShell.
4. Return only the issue number and URL.

Notes:

- File at discovery time, not at the end of the plan. Issue creation is not a
  commit, so there is nothing to stage.
- If the concern is resolved before the plan concludes, close the issue with
  `gh issue close <n> --comment "<why>"` — do not delete it, the history is the
  point.
- GitHub Actions runs are also visible via `gh run list` /
  `gh run view <run-id> --log-failed`; strip ANSI before grepping.

## Grounding the plan

- **Ground plan decisions in official docs** — where the plan relies on framework behavior (Spring Boot / Spring Security / Spring Data JPA, Angular Material, Apache POI Excel parsing, Flyway / PostgreSQL, WiqsServer ASP.NET Core), ground the implementation plan and strategy in official documentation and examples fetched via the microsoft-learn MCP server (`microsoft_docs_search`, `microsoft_code_sample_search`, `microsoft_docs_fetch`). This is **Gate G1**: it runs immediately after the plan is committed and before any implementation, and anything it surfaces that needs a decision is escalated to the human — never resolved by assumption.
- **Navigate source with code-memory MCP** — when exploring the codebase for planning or design, prefer the code-memory MCP tools (symbols, relationships, call graphs, impact analysis) over ad-hoc grepping.
- **Assess blast radius** — for every plan, identify and state the blast radius: what files/components/symbols each change affects, what depends on them (downstream callers), and which tests cover them. Include this in `docs/TODO.md` so the human can judge risk before execution.

## Commit Message Format

```
<short summary in imperative mood>

<optional body explaining why and what changed>
```

Example:
```
Fix flexible Excel header matching for CRLF-wrapped headers

Report headers wrapped across cells by Excel (e.g. "Resource\r\nDisplay\r\nName")
failed the exact-match lookup and rejected the file with a row-0 header error.
Header matching now strips whitespace and compares case-insensitively, so
CRLF-wrapped and shuffled header orders map to the six required columns.
```

## Workflow

1. Complete the step (code change + the smallest meaningful verification — see AGENTS.md → Testing Guidance)
2. `git status` to see changed files
3. `git diff` to review changes
4. Capture deferred work — scan the step for anything worth doing later (limitations, follow-ups, concerns). If found, open a GitHub issue for it now via a subagent; never hold it in memory.
5. `git add <specific files>` — stage only the files for this step
6. `git commit -m "<message>"`
7. Report to the human what was committed (without pushing)
8. Continue to next step

## When a test fails

A failing test is an opportunity to assess the root cause properly — not a signal to patch around it. Before changing anything:

1. **Diagnose first** — determine whose expectations are wrong:
   - Did **our change** introduce a regression (a bug in the code)?
   - Did the **design/contract** change (feature, API, or behavior intentionally changed, so the old expectation no longer holds)?
   - Is the **test stale** — asserting behavior we superseded?
2. **If our code change is the problem**, fix the code — don't edit the test to make it pass.
3. **If the expectation legitimately changed** (design change, new contract), update the test to assert the new correct behavior and record the intent in the commit message / `docs/TODO.md`.
4. **Don't rush quick fixes** — approach it the right engineering way, even if it takes longer.

## Test coverage balance

- **Existing tests are the consistency guardrail** — as we progress and sometimes break things, the existing suite is how we catch regressions and stay consistent. When adding something new, keep existing tests green unless a contract deliberately changed (see "When a test fails").
- **Cover new work with good-enough tests** — enough to verify the new behavior and its edge cases, but not so much that tests become brittle or a maintenance chore. Prefer extending the existing suite over adding redundant new tests.
- **Keep tests focused and readable** — assert behavior, not implementation details; over-testing internals makes refactors painful.

## What NOT to do

- Do NOT use `git push` at any point
- Do NOT amend previous commits unless explicitly asked
- Do NOT use interactive rebase or squash
- Do NOT commit secrets, keys, or credentials
- Do NOT commit generated files or build artifacts
- Do NOT create a duplicate issue when an open one already covers the concern — comment on it instead
- Do NOT open issues for work already done in the current plan; only for what is genuinely deferred