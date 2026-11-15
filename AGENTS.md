# Agent Notes

Start by reading [IDEA.md](docs/IDEA.md) (the technology-free north star) and then [POC.md](docs/POC.md) (what we are building first, and the build order). [README.md](README.md) has the overview.

This repo is pre-code and early. Keep changes small, sequential, and easy to verify. Follow the build order in POC.md section 11 and do not skip ahead.

## Two Different AGENTS.md Files

Do not confuse them:

- **This file** instructs agents working on the WorktreeIQ repo itself.
- **`samples/<domain>/AGENTS.md`** is *generated* by the harness from that domain's pack and instructs the worker that runs inside that domain folder. Never hand-edit a generated file. Change the pack and regenerate.

## What WorktreeIQ Is

A domain-free harness that launches an AI worker into a domain folder and enforces that domain's structure. Harness plus domain folder is a working agent (POC.md section 3).

It is NOT a CMS, a chat UI, a general data pipeline framework, or a vector database. It is NOT tied to any domain.

## Hard Rules

1. **The harness stays domain-free.** No page type, field name, folder name or rule from any domain (delivery included) may appear in harness code. Domain knowledge lives only in the domain folder's pack. If you need a domain fact in the harness, the pack format is missing a feature; fix the format instead.
2. **Enforcement is structural, not instructional.** The worker gets a small tool set and the gatekeeper sits behind every write. Never rely on a prompt to keep the worker inside the rules.
3. **Instructions and enforcement come from one source.** The generated AGENTS.md and the validators must both be derived from the pack. A test must fail if they can diverge.
4. **The worker never writes harness-owned files** (`index.md`, `log.md`, run records, generated AGENTS.md) and never touches `raw/`.
5. **Wiki files are the source of truth.** Any database, index or serving layer is a projection of the files and can be rebuilt from them.
6. **A failed run leaves the wiki untouched.** Changes are accepted only after the per-write and whole-wiki checks pass.
7. **Missing is not invented.** The explicit "unknown" and "not in source" values must stay valid, visible, and distinct from absent. Never loosen a rule or fill a value just to make a check pass.
8. **Rejection messages are a product feature.** Every rejection must say which rule failed, on which page and field, and what would satisfy it. Test message quality, not only the reject/accept outcome.
9. **Synthetic data only** in samples and tests. No real personal data.

## Forbidden

- Custom LLM clients and custom embedding pipelines. Use the platform's standard chat and embedding abstractions.
- Custom vector databases.
- Hard-coding domain content in the harness (rule 1).
- Letting the model write directly to the wiki folder, bypassing the gatekeeper.
- Hand-editing generated files.

## Tech Stack (working assumption, not yet decided)

.NET is the working assumption, to reuse the Semantics library carried over from WorkplaceIQ and to match existing tooling. **Confirm in an ADR before writing the first project.** If the ADR changes the stack, update this file and the code-style section below.

## Project Structure (planned)

```
README.md  AGENTS.md
docs/IDEA.md                 north star (technology-free)
docs/POC.md                  first proof of concept
docs/adr/                    decisions
src/
  WorktreeIQ.Packs/        pack model, loader, validators, AGENTS.md generator
  WorktreeIQ.Harness/      launch, plan, gatekeeper, run records
  WorktreeIQ.Reader/       ask mode
  WorktreeIQ.Semantics/    HTML to Markdown, carried over from WorkplaceIQ
  WorktreeIQ.Cli/          ingest, ask, check
tests/
samples/
  delivery/                  first domain folder: pack/, raw/, wiki/, .workfolder/
```

Create projects only when their build step in POC.md is reached.

## Domain Folder Layout

```
<domain>/
  AGENTS.md          generated, never hand-edited
  pack/              pack.md, types/*.md, rules/*.md
  raw/               immutable sources
  wiki/              pages, plus harness-owned index.md and log.md
  .workfolder/       harness state: plans, run records, review queue
```

## Testing

- The gatekeeper and validators must be testable **without a model**. Drive them with hand-written good and bad pages and with a scripted fake worker that replays recorded plans and writes.
- Cover POC.md acceptance scenarios A to G as tests wherever they can be deterministic. Real-model runs are evaluation, not unit tests; keep them separate.
- Test the "second pack" scenario (G) as early as it is cheap: it is the guard against domain leakage into the harness.
- Keep tests split by behavior rather than in one large file.
- Naming: `Method_Scenario_ExpectedBehavior`. Pattern: Arrange-Act-Assert, no comments needed.

## ADR Style

ADRs live in `docs/adr/`, numbered, in the format: Context, Decision, Rationale, Consequences, Alternatives considered.

ADRs describe **what** and **why** at an architectural level, without pinning method names, parameter types or class signatures that will drift. Include intent, constraints and tradeoffs. If an ADR contradicts the implementation, update the ADR toward the abstract intent; the implementation is the source of truth for specifics.

Planned first ADRs (POC.md section 11):

1. Wiki files are the source of truth; any serving layer is a projection.
2. The harness is domain-free; the pack travels with the domain folder.
3. Stack and model-access abstraction (confirm the working assumption above).

Other open decisions are listed in POC.md section 12. Record each as an ADR when decided, and update that list.

## Working Style

- Make the smallest change that moves the current build step forward, and verify it before the next.
- IDEA.md stays technology-free. Technology choices go in ADRs and POC.md, never IDEA.md.
- When something in IDEA.md or POC.md turns out to be wrong or unworkable, say so and propose an edit to the document rather than quietly working around it.
- Keep POC.md's "Decided / Leaning / Open" split honest: move items between them as decisions are made.
- Prefer observations first, then proposals, then ADRs, as the WorkplaceIQ docs do, when a design question needs discussion.

## Relationship to WorkplaceIQ

WorkplaceIQ (https://github.com/khurram-uworx/WorkplaceIQ) is a separate project. Do not import its EF Core store, tag helpers, metrics platform or demo app. Carry over only what POC.md names: the Semantics library, the conventions, and later the SignalFlow ideas. Copy what is needed and own it here; do not add a dependency on the WorkplaceIQ repo.

## Build And Test

To be filled in when the first project exists. Record the exact commands here, and anything that behaves unreliably in the dev environment, as the WorkplaceIQ AGENTS.md does.
