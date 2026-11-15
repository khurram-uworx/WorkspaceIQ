# WorktreeIQ

A harness that launches an AI worker into a **domain folder** and guarantees the wiki it builds follows that domain's structure.

Think of how a coding agent works: the harness (Claude Code, Codex, Copilot) brings identity and controls, the repo brings context and rules, and together they become a working agent. WorktreeIQ applies the same pairing to knowledge work:

> **WorktreeIQ harness + a domain folder = a working agent for that domain.**

Same harness, delivery folder: a delivery agent. Same harness, HR folder: an HR agent.

## The Idea in Brief

Letting an LLM turn raw data into wiki pages freely produces drift: pages in random places, duplicate entities, missing fields, guesses mixed with facts. Instructions alone don't prevent it.

WorktreeIQ takes the MVC approach instead. Each domain defines a fixed set of page types, each with one home, one shape, and its own required and optional fields. A **gatekeeper** checks every write and rejects anything off-structure with a specific reason, so the worker can correct itself. The instructions tell the worker what is expected; the harness makes sure it happens.

Pages stay plain Markdown with frontmatter, like skills. Raw sources stay immutable and every page traces back to them.

Read [IDEA.md](docs/IDEA.md) for the full north star (technology-free) and [POC.md](docs/POC.md) for the first proof of concept.

## Status

**Pre-code.** This repository currently holds the design documents. The first POC is planned in [POC.md](POC.md); nothing is implemented yet, and the stack is a working assumption to be confirmed in an ADR.

## How It Fits Together

```
        raw sources (immutable)
                 │
                 ▼
   ┌──────────── harness ────────────┐
   │  launch → plan → gatekeeper →   │   ingest mode: worker writes
   │  whole-wiki check → accept      │   within the pack's rules
   └──────────────┬──────────────────┘
                  │ reads rules from
                  ▼
          domain folder
   pack/ (rules)   wiki/ (pages)   raw/   AGENTS.md (generated)
                  ▲
                  │ read-only
              ask mode (reader)
```

| Piece | Role |
|---|---|
| **Harness** | Domain-free. Launches the worker, requires a plan, checks every write, runs the whole-wiki check, owns the index and log. |
| **Domain folder** | Everything domain-specific: the pack, the generated AGENTS.md, raw sources, wiki pages. |
| **Pack** | Declarative rules: page types, locations, fields, conditional rules, body requirements, link rules. |
| **Ingest mode** | The worker may write, only through the gatekeeper. |
| **Ask mode** | Read-only. The reader answers questions using the typed structure first, free text second. |

## Repository Layout (planned)

```
AGENTS.md      conventions for agents working on this repo
docs/IDEA.md   north star
docs/POC.md    first proof of concept
docs/adr/      architecture decisions
src/           harness, pack model, reader, CLI (not yet created)
samples/
  delivery/    the first domain folder: pack, raw, wiki
tests/
```

## First Domain: Engineering Delivery (Scrum of Scrums)

`samples/delivery` is the first sample domain: teams, boards, sprints, work items, cross-team dependencies and impediments, built on synthetic Jira snapshots. The wiki is the cross-team layer above the board, not a copy of it, so only work that is blocked, crosses a team boundary or was carried over earns a page.

It exists to prove the idea and to keep the harness honest, since anything delivery-specific that leaks into the harness is a bug.

## Relationship to WorkplaceIQ

[WorkplaceIQ](https://github.com/khurram-uworx/WorkplaceIQ) is a separate, database-backed content platform that continues at its own pace. WorktreeIQ borrows from it selectively (the Semantics HTML-to-Markdown library, repo conventions, the ADR format) and does not depend on it. A work folder could later be indexed into WorkplaceIQ as one reader, but that is outside the POC.

## Getting Started

Not applicable yet. Once code exists, this section will hold the quick start and the build and test commands. Until then, start with [IDEA.md](docs/IDEA.md), then [POC.md](docs/POC.md), and follow the build order in POC.md section 11.
