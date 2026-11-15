# IDEA: A Structured Enterprise Wiki Maintained by an AI Worker

## 1. The Idea in One Paragraph

Build an enterprise knowledge base made of plain, readable pages that an AI worker creates and maintains from raw incoming data. Unlike an open-ended "let the AI write whatever it wants" approach, the wiki follows a **strict, domain-specific structure**: every kind of knowledge has exactly one home, one shape, and a defined set of required and optional fields. The AI is a worker inside a controlled workspace, and a **gatekeeper** (the harness) guarantees the rules are followed. Instructions tell the AI what is expected; the gatekeeper makes sure it happens.

## 2. The Problem

Letting an AI freely turn raw data into wiki pages leads to predictable decay:

- Pages appear in arbitrary places, with arbitrary names and shapes.
- The same entity (a team, a work item) ends up on several pages, each slightly different.
- Important fields are missing on some pages and present on others.
- Facts, guesses and invented details are mixed together with no way to tell them apart.
- Nobody can trace a statement back to where it came from.
- The reader (human or AI) cannot rely on the structure, so every query becomes a fresh guessing exercise.

Instructions alone ("please put every team in the teams folder") drift over time and across runs. A model can ignore, misread or forget them.

## 3. The Core Analogy

Think of how a web application framework works: there are only models, views and controllers, and each goes in a known place. Developers cannot invent a fourth category on a whim, and the framework will complain if something is in the wrong spot.

This idea applies the same discipline to knowledge:

- A **fixed set of page types**, each with a designated location.
- A **defined shape** for each page type.
- A **gatekeeper** that rejects anything off-structure, the way a framework rejects a malformed project.

It also resembles how a coding agent works inside a repository: it is launched into a working folder, reads the project's instructions, proposes a plan, and then makes changes. Here, the "project" is the wiki and the "code" is knowledge pages.

## 4. Guiding Principles

1. **Structure over freedom.** The AI fills in a well-defined structure; it does not design one.
2. **Enforcement over instruction.** Rules are checked by the system, not merely requested of the AI.
3. **Provenance over assertion.** Every statement traces back to an immutable raw source.
4. **Facts and judgments are kept apart.** What the source said is distinguished from what the AI concluded.
5. **Honest gaps over invented fillers.** "Not in source" and "unknown" are valid, visible answers. Fabricating a value to satisfy a rule is never acceptable.
6. **One entity, one page.** Duplicates are a defect, not a side effect.
7. **Plan, then act.** The AI proposes what it intends to create or change before anything is written.
8. **Domain rules are data, not code lore.** Each domain describes its own structure and rules in a declarative pack that can be reviewed and changed by non-engineers.

## 5. The Actors

### 5.1 The Ingestion Worker (writer)
An AI that is given a workspace and a batch of raw material. It proposes a plan, then creates or updates pages according to the domain's rules.

### 5.2 The Gatekeeper (the harness)
The system that launches the worker, confines it to the workspace, and checks every action against the domain's rules. It can refuse an action, explain precisely why, and let the worker correct itself. It also owns certain files the AI is never allowed to touch (such as the master index and the change log).

### 5.3 The Reader (chat / query experience)
A separate application that answers questions using the wiki. It has read-only access. Because pages are typed and structured, it can answer precise questions by looking at fields ("all work items blocked on a single team") before falling back to free-text understanding.

### 5.4 The Domain Owner (human)
A subject-matter person who defines or approves the domain pack and reviews flagged items. They do not need to write any program.

## 6. The Domain Pack

A **domain pack** is the complete, declarative description of one domain's wiki. It contains:

- **Page types**: what kinds of pages exist (and no others).
- **Locations**: the one place each page type may live, and how it must be named.
- **Fields**: for each type, which fields are required, which are optional, and what values are allowed.
- **Conditional rules**: "if X is true, then Y becomes required." (Example from medicine: a heart patient's visit must include a blood pressure reading. Example from engineering delivery: a work item that is in progress must have an owner and an estimate.)
- **Body requirements**: sections a page must contain, and which sections may only be added to rather than rewritten.
- **Relationship rules**: which pages must link to which, and that links may not point to nothing.
- **Instructions for the AI**: a human-readable guide generated from the same rules, so that what the AI is told and what the gatekeeper enforces can never disagree.

Different domains (medical, engineering delivery, HR, incident management) are simply different packs on the same machinery.

## 7. How a Run Works

1. **Launch.** The gatekeeper starts the worker in a fresh workspace containing the current wiki, the domain pack's instructions, and the new raw material.
2. **Plan.** The worker states what it intends to do: which pages to create, which to update, which entities it believes already exist. The gatekeeper checks the plan against the rules. Optionally, a human approves it.
3. **Act.** The worker writes pages. Every write is checked immediately: right location, right name, right fields, right shape. Violations are rejected with a clear, specific explanation, and the worker fixes and retries.
4. **Whole-wiki check.** Some rules can only be judged across pages (for example, "a cross-team dependency must be recorded on both teams' pages"). These are checked before the run's changes are accepted.
5. **Accept or reject.** Changes become part of the wiki only if all checks pass. Anything uncertain is surfaced for human review rather than silently accepted.
6. **Record.** The gatekeeper updates the index and the change log, which the AI never writes itself.

## 8. Handling the Hard Parts

### 8.1 Missing information
Strict required fields tempt an AI to make things up. The structure therefore includes an explicit, valid way to say "the source does not contain this" or "unknown". These are accepted by the rules but collected for review, so a real gap is distinguishable from lazy work.

### 8.2 Fact versus inference
Pages distinguish **observations** (backed by a source excerpt) from **judgments** (the AI's conclusions, such as "at risk" or "high impact"). Judgments carry a confidence and point to the observations that support them. The reader can then say "inferred from these three items" instead of presenting a guess as truth.

### 8.3 Verifying evidence
Where a page quotes a source, the gatekeeper can confirm the quote actually exists in the raw material. Fabricated quotes are caught mechanically.

### 8.4 Duplicates and identity
Incoming data repeats constantly and refers to the same thing under many names. Every entity has a stable identity, and ingestion must first decide "is this an existing entity or a new one?" and justify the answer. Re-ingesting the same raw item must update or skip, never duplicate.

### 8.5 Updates
Existing pages are updated, not rewritten at will. Structured fields may be changed; narrative sections that represent history are append-only. This keeps changes reviewable and prevents silent loss of earlier knowledge.

### 8.6 Raw sources are sacred
The original material is stored unchanged and referenced by every page that used it. The wiki is a derived view; the raw data remains the ground truth.

## 9. Pilot Domain: Engineering Delivery (Scrum of Scrums)

Chosen because software engineers are the natural first audience, the raw input (board exports from a tracker such as Jira) is structured enough that provenance can be checked exactly, and cross-team delivery has rules with real conditions. It carries far less legal weight than medicine, and the data is synthetic.

The wiki is **not a mirror of the board tracker**. The board is where the work lives; the wiki holds the Scrum-of-Scrums layer above it: the teams, what each committed to, what crosses team boundaries, and what is slowing delivery down. A copied board would be a stale duplicate, so only records that matter across teams earn a page of their own.

### 9.1 Page types and homes

| Page type | Lives in | One page per |
|---|---|---|
| Team | `teams/` | team |
| Board | `boards/` | board (one tracker board, scrum or kanban) |
| Sprint | `sprints/<team>/` | sprint |
| Work Item | `work-items/<team>/` | issue that matters beyond its own team |
| Dependency | `dependencies/` | cross-team need where one team's work gates another's |
| Impediment | `impediments/` | factor currently slowing delivery, with its impact |

Nothing else may exist. An index and a change log are owned by the gatekeeper.

A raw board export does not have to become a page. It updates the pages it touches. When a raw record deserves a page of its own is the **admission rule**, and the pack must state it (§12, question 8). In the pilot, admission is: an issue gets a page once it is blocked, part of a cross-team dependency, or carried over from a previous sprint.

### 9.2 Illustrative fields

- **Team**: name, tracker project key, delivery lead, board links. Optional: parent group, member roster.
- **Board**: name, tracker board id, team, board type (scrum or kanban), column-to-status mapping, last snapshot reference.
- **Sprint**: team, name, start date, end date, state (planned, active, closed), goal. Its items are links, not a stored count.
- **Work Item**: key, summary, team, sprint, status, assignee. Optional: estimate, item type, links (blocked by, depends on). Required when relevant: a disposition if its sprint closed without it being done.
- **Dependency**: from item, to item, requesting team, fulfilling team, state, owner on each side, needed-by date. Optional: how long it has been in its current state.
- **Impediment**: affected items and teams, category, opened date, owner, state. Its impact is a judgment with a confidence and cited snapshots.

### 9.3 Illustrative rules

- A work item in the active sprint that is in progress must have an assignee and an estimate (or an explicit "unknown" that is flagged).
- A blocked work item must link to a dependency or an impediment that names an owner.
- A cross-team dependency must be recorded on both teams' pages and name an owner on each side.
- A dependency whose state has not changed across several snapshots is at risk and must be raised as an impediment.
- A work item in a closed sprint that is not done must carry a disposition (carried over, dropped, split). Work may be dropped openly; it may not vanish.
- A work item has exactly one page, keyed by its tracker key. A second page for the same key is a duplicate defect.
- When two snapshots disagree about current state, the newer snapshot wins, and the earlier value stays in the item's history and is flagged. Nothing is silently overwritten.
- Impact and risk are judgments and must cite the snapshots that support them, with a confidence. A "bottleneck team" is not a judgment: it is computed from the direction of open dependencies, so it cannot drift from the pages that produce it.

### 9.4 Reader scenarios

- "What is blocked across all teams right now, and who owns each unblock?"
- "Which team is the bottleneck this sprint, and which dependencies make it so?"
- "What changed on Team X's board since the last snapshot?"
- "Which items have slipped from sprint to sprint without a disposition?"
- "Which teams are carrying unestimated work into the active sprint?"
- "Which dependencies have been waiting on the other team the longest?"

## 10. What This Idea Is Not

- Not a free-form note-taking assistant.
- Not a search index that hides structure inside opaque storage; the pages are human-readable.
- Not tied to one domain; engineering delivery is only the first pack.
- Not a replacement for human judgment on flagged items.

## 11. Success Looks Like

- Anyone can predict where a piece of knowledge lives without searching.
- No duplicate entities after repeated ingestion of overlapping data.
- Every claim can be traced to a raw source in one step.
- Gaps and AI judgments are visible as such.
- Adding a new domain is a matter of writing a pack, not rebuilding the system.
- The reader gives noticeably more precise answers than it would over unstructured pages.

## 12. Open Questions

1. How much of a domain pack can a non-engineer author comfortably, and how are changes to a pack reviewed?
2. When the rules change, how are existing pages migrated or re-validated?
3. What is the right granularity for human approval: every plan, only flagged items, or only on request?
4. When two snapshots of the same board disagree, should the newer one win, should both be shown, or should the disagreement be flagged?
5. How are personal data and retention handled for board exports, which contain assignees and comments, and what access do we need to the source system?
6. How is access controlled when different readers should see different parts of the wiki?
7. How do we measure whether structure actually improves the reader's answers compared to a free-form wiki?
8. When does a raw record deserve a page of its own (the admission rule), and how does a pack declare that so it can be enforced rather than requested?

## 13. Suggested First Milestone

Define the engineering delivery domain pack in full (page types, fields, conditional rules, body requirements), prepare a small set of synthetic board snapshots, and demonstrate three things end to end: the worker producing compliant pages, the gatekeeper rejecting a deliberately non-compliant attempt with a useful explanation, and the reader answering a structured question that would be hard to answer over free-form notes.
