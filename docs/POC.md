# POC: WorktreeIQ

> Companion to [IDEA.md](IDEA.md), which stays the technology-free north star.
> This document records where we stand and what the first proof of concept looks like.
> Items marked **(proposed)** are not decided yet.

## 1. Purpose of the POC

Prove one claim from IDEA.md end to end, on a small but realistic domain:

> A generic harness plus a domain folder can produce a wiki that follows a strict structure, because the structure is **enforced**, not merely requested.

The POC is successful if we can demonstrate the three things named in IDEA.md section 13 (compliant worker output, a useful rejection of non-compliant output, a structured question answered by the reader), plus the properties in section 9 below.

## 2. Where We Stand

### Decided

| Topic | Decision |
|---|---|
| Relationship to WorkplaceIQ | WorkplaceIQ stays as it is and moves at its own pace. WorktreeIQ is a new, separate project. |
| Carry-over | Selective, not a full clone: the Semantics library, repo conventions (AGENTS.md style, ADR format, skills), and later the SignalFlow ideas. The EF store, tag helpers, demo app and metrics platform are left behind. |
| Source of truth | Markdown files with frontmatter in a folder. Any database or serving layer is a projection of the files, never the reverse. |
| Working-agent model | Harness + domain folder = working agent (section 3). |
| Harness is domain-free | Everything domain-specific lives in the domain folder. |
| Pilot domain | Engineering delivery (Scrum of Scrums), with synthetic board data. Familiar to our audience, and the input is structured. |
| POC input boundary | Board snapshots exported from Jira, dropped into `raw/`. No live connector, OAuth or webhook in the POC. |
| Enforcement | A gatekeeper checks writes; instructions alone are not trusted. |
| Modes | Ingest (worker writes within the rules) and ask (read-only reader). |
| Name | WorktreeIQ. |

### Leaning (proposed)

- The worker never gets raw filesystem access. It gets a small set of tools (read, search, propose plan, write page, flag for review), and the gatekeeper sits behind every write. That makes enforcement structural rather than a hook that could be bypassed.
- Stack: .NET, matching the author's other work and allowing reuse of the Semantics library. Model access through the existing chat and embedding abstractions; no custom LLM client. This is an assumption to confirm in an ADR, not a commitment.
- Pack files are Markdown with frontmatter, like skills: the frontmatter is the machine-readable rule, the body is the human guidance, and the generated AGENTS.md is assembled from both.

### Open

See section 12.

## 3. The Working-Agent Model

A coding agent is a harness (identity: how it plans, which tools it has, how permissions and checks work) launched inside a repo (context: AGENTS.md, skills, hooks, the code). Together they form a working agent. The same pairing applies here.

| Coding agent | WorktreeIQ |
|---|---|
| Claude Code / Codex / Copilot | WorktreeIQ harness |
| The repository | A domain folder (a "work folder") |
| AGENTS.md / CLAUDE.md | Generated AGENTS.md, assembled from the pack |
| Hooks, permissions | Gatekeeper rules from the pack |
| Skills | Page-type guidance in the pack |
| Plan mode | Plan step, validated before writing |
| Read-only mode | Ask mode |
| Same tool, different repo | Same harness, different domain folder = different working agent (delivery agent, HR agent) |

Consequences:

- The harness repo contains **no** domain knowledge. Delivery assumptions leaking into the harness is a bug.
- The pack is versioned together with the pages it governs, which gives a natural starting point for the "rules changed, now what?" question.

## 4. Proposed Repository Layout

```
WorktreeIQ/
  README.md
  AGENTS.md                  conventions for agents working on this repo
  docs/
    IDEA.md                  north star (technology-free)
    POC.md                   this document
    adr/                     decisions, in the Context / Decision / Rationale /
                             Consequences / Alternatives format
  src/
    WorktreeIQ.Packs/      pack model, loader, validators, AGENTS.md generator
    WorktreeIQ.Harness/    launch, plan, gatekeeper, run records
    WorktreeIQ.Reader/     ask mode (thin in the POC)
    WorktreeIQ.Semantics/  carried over from WorkplaceIQ (HTML to Markdown)
    WorktreeIQ.Cli/        entry point: ingest, ask, check
  tests/
  samples/
    delivery/                the first domain folder (section 5)
```

Later domains (for example an incident log) would be added as further folders under `samples/`, or live in their own repos.

## 5. Anatomy of a Domain Folder: `samples/delivery`

```
samples/delivery/
  AGENTS.md                  generated from the pack; never hand-edited
  pack/
    pack.md                  domain name, version, global rules
    types/
      team.md                one file per page type
      board.md               (location, naming, fields, body sections,
      sprint.md               update rules, guidance for the worker)
      work-item.md
      dependency.md
      impediment.md
    rules/
      cross-page.md          whole-wiki conditional rules
  raw/
    <snapshot-id>/           immutable Jira export: boards, sprints, issues
  wiki/
    teams/  boards/  dependencies/  impediments/
    sprints/<team>/  work-items/<team>/
    index.md  log.md         harness-owned; the worker cannot write these
  .workfolder/               harness state: plans, run records, review queue
```

Each snapshot folder in `raw/` is one export, stored unchanged with its export timestamp. Pages cite the snapshot and the record id they came from, so a later snapshot can be compared against the previous one and provenance can be checked exactly.

A page-type file in `pack/types/` carries, in its frontmatter: home folder, filename pattern, required fields, optional fields, allowed values, which sections are append-only, and which links are mandatory. Its body is the guidance the worker reads. **(proposed)**

The wiki is the cross-team layer, not a copy of the board. A raw issue becomes a work-item page only when it is blocked, part of a cross-team dependency, or carried over (the admission rule). Everything else stays in `raw/` and is reachable from the team and sprint pages. **(proposed; see section 12)**

## 6. How a Run Looks in the POC

Commands are illustrative **(proposed)**:

```
WorktreeIQ check  samples/delivery          # validate the whole wiki against the pack
WorktreeIQ ingest samples/delivery          # launch the worker on new raw material
WorktreeIQ ask    samples/delivery "..."    # read-only reader
```

An ingest run:

1. **Launch.** Harness loads the pack, regenerates AGENTS.md, opens a run workspace, and gives the worker the new snapshot.
2. **Plan.** The worker submits a structured plan: pages to create or update, and for each entity whether it matches an existing page (with justification). The gatekeeper validates the plan. In the POC, approval is a simple confirmation in the terminal.
3. **Act.** The worker writes through the page-write tool. Each write is checked and either accepted or rejected with a precise reason; the worker can retry.
4. **Whole-wiki check.** Cross-page rules are evaluated over the changed wiki.
5. **Accept or reject.** Failing runs leave the wiki untouched. Anything unresolved goes to the review queue.
6. **Record.** Harness updates `index.md` and `log.md` and stores the run record.

### Checks in scope for the POC

**Per write**
- Path is inside a declared folder and matches the filename pattern for its type.
- Page type is one of the declared types.
- Required fields present, values valid; explicit "unknown" or "not in source" accepted but flagged.
- Required body sections present; append-only sections not rewritten.
- Links point to existing pages (or pages created in the same run).
- Quoted evidence appears verbatim in the referenced raw source.
- Harness-owned files are untouchable.

**Per run**
- Conditional rules (for example, a work item in the active sprint that is in progress needs an assignee and an estimate or a flagged unknown).
- Dependency rules (for example, a cross-team dependency must be recorded on both teams' pages and name an owner on each side).
- No duplicate entities; identity rules hold.
- Judgment fields cite supporting snapshots and a confidence.

## 7. Delivery Pack Content for the POC

All six page types are defined and exercised by the two snapshots, so the "only these folders" rule is proven by rejecting anything written outside them (scenario B), not by leaving a type unused.

Fields and rules start from IDEA.md section 9 and are refined while building. The rules worth building first are the ones that need history or cross-page judgement: an in-progress item needs an owner and an estimate, a closed sprint may not drop work silently, a dependency must exist on both sides, and a dependency that has not moved across snapshots must be raised as an impediment.

Judgments (impediment impact, risk) carry a confidence and cite the snapshots that support them. The bottleneck team is derived from dependency fields instead, so it cannot be asserted without the pages that justify it.

## 8. Sample Data

A fictional product organisation on Jira Cloud: four teams (Payments, Platform, Mobile, Insights) across four projects, with one team on a kanban board instead of sprints. Two synthetic snapshots three weeks apart, and roughly 15 to 25 issues so the sample stays readable.

Only the issues that meet the admission rule become pages; the rest stay in `raw/`. This is deliberate: it tests that the worker projects the board rather than mirroring it.

It is deliberately seeded with traps:

- A sprint closed with an item that is not done and carries no disposition.
- A cross-team dependency recorded by the requesting team only.
- A dependency whose state has not changed between the two snapshots, which must raise an impediment.
- An in-progress item with no estimate (the "unknown" path).
- A project renamed between snapshots: same key, new name, one team page.
- An issue moved from one project to another, so the old page must not survive beside the new one.
- The same snapshot delivered twice.
- Two boards for one team (a scrum board and a kanban board) that both contain the same issue.
- A tempting inference ("Platform is slow") with no evidence behind it.
- A dependency pointing at a team that is not yet a page.
- Two snapshots that disagree about an item's status, so the newer value wins and the earlier one is flagged rather than lost.

## 9. Acceptance Scenarios

| # | Scenario | Passes when |
|---|---|---|
| A | Compliant ingest | A snapshot produces pages in the right folders, passing every check, each citing its snapshot and record id |
| B | Rejection quality | A wrong folder, a missing assignee on an in-progress item, an invented quote, and a dead link to a team page are each rejected with a message specific enough for the worker to self-correct |
| C | Identity and dedup | Re-ingesting the same snapshot changes nothing; a renamed project keeps one team page; a moved issue keeps one work-item page |
| D | Honest gaps | A missing estimate is recorded as unknown, accepted, and visible in the review queue |
| E | Whole-wiki rule | A dependency recorded on one side only, or a closed sprint dropping an item with no disposition, causes the run to be rejected and the wiki left untouched |
| F | Ask mode | "Which team is the bottleneck this sprint, and which dependencies make it so?" is answered from dependency and team fields, with the supporting pages cited |
| G | Domain-free harness (stretch) | A second, tiny pack (for example an incident log) runs on the same harness with no harness changes |

## 10. Scope

**In:** the harness core, the pack loader and validators, the AGENTS.md generator, CLI, the delivery sample (pack plus synthetic Jira snapshots), a thin reader, tests.

**Out for the POC:** any UI; multi-tenant and access control; live Jira or Linear connectors, OAuth, webhooks or scheduled sync; vector search; a database or serving layer; integration with WorkplaceIQ; pack migration tooling; a polished human-approval experience.

## 11. Build Order

Small, sequential, each step verifiable (matching the working style in the WorkplaceIQ AGENTS.md):

1. ADR-01 (wiki files are the source of truth) and ADR-02 (harness is domain-free; pack travels with the folder).
2. Pack model and loader; write the delivery pack.
3. Validators: per-page checks, then link checks, then the whole-wiki check. Drive them with hand-written good and bad pages. `WorktreeIQ check` works before any model is involved.
4. AGENTS.md generator, with a test that proves the generated instructions and the enforced rules come from the same source.
5. Gatekeeper-mediated page-write tool and run workspace (accept or reject, leave the wiki untouched on failure).
6. Scripted fake worker that replays recorded plans and writes. This lets scenarios B, C, D and E run in tests with no model, and keeps them deterministic.
7. Real worker through the chat abstraction; plan step; review queue; index and log.
8. Thin reader for scenario F.
9. Optional: a second raw-snapshot format (for example Jira CSV); second pack for scenario G. The Semantics HTML-to-Markdown front door is no longer needed for the pilot and moves to the back of the queue.

## 12. Open Decisions

1. Final stack and the model-access abstraction (confirm in an ADR).
2. Exact pack file format and how conditional rules are expressed (declarative expressions, or a small code escape hatch for the hard ones).
3. Plan approval granularity: every plan, only flagged items, or on request.
4. Representation of conflicting snapshots: newest wins, show both, or flag.
5. Identity scheme for teams, work items and snapshots (so re-ingestion is idempotent and a renamed project or a moved issue still resolves to one page).
6. How a worker run is isolated and cleaned up on failure.
7. Where the review queue lives and how a human resolves items.
8. How to evaluate whether structure actually improves reader answers compared to a free-form wiki.
9. The admission rule: which raw issues earn a page (mirror everything, or only blocked, cross-team and carried-over work), and how the pack declares and enforces it.
10. How many snapshots are kept, and whether "what changed since last sync" is computed from the raw snapshots, the change log, or both.

## 13. Risks and Assumptions

- **Required fields invite fabrication.** The explicit "unknown" path must be easy and visibly distinct from missing work; scenario D tests it.
- **Rejection messages are the product.** A worker that gets vague errors will thrash. Scenario B is about message quality, not just rejection.
- **Model non-determinism.** Gatekeeper behavior must be testable without a model (build step 6), with real-model runs treated as evaluation, not unit tests.
- **Cost and latency.** Plan, then act, then retry loops multiply model calls. Measure early.
- **Our own domain can flatter the design.** We are the audience this time, so delivery assumptions are easier to mistake for general ones. Scenario G matters more, not less.
- **"Sync" invites scope creep.** The POC boundary is snapshots dropped into `raw/`. A connector, OAuth flow or webhook is a different project and must be argued for in an ADR.
- **A mirrored board would be a stale duplicate.** The admission rule is what keeps the wiki at the cross-team layer; if it is skipped, the sample becomes a worse Jira.
- **Personal data in board exports.** Assignees and comments are people data. Synthetic data only in the POC; retention and access to real exports are out of scope here.

## 14. Later: Relationship to WorkplaceIQ

Not part of the POC, but the mapping is natural if wanted: page types to containers, frontmatter to item metadata, links to relationships, judgments to classifications, the accepted wiki indexed for search and chat. WorkplaceIQ could become one reader of a work folder without either project changing its core.
