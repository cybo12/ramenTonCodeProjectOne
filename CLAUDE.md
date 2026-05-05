# MTG Circle

Web platform for a group of friends to manage Magic: The Gathering collections, propose trades, and build decks intelligently. The product backlog lives in Notion ("🃏 MTG Circle — Backlog"); design docs and implementation plans live in `docs/superpowers/`.

## Tech Stack

| Layer | Tech |
|---|---|
| Backend | Python 3.14, FastAPI, SQLModel, Alembic, PostgreSQL 16, uv |
| Frontend | Next.js, Tailwind, NextAuth (planned, not yet scaffolded) |
| Infra | Docker Compose, Redis, Adminer |
| External APIs | Scryfall (card data), CardMarket (prices, Phase 3) |

## Monorepo Layout

- `apps/backend/` — FastAPI service. Has its own `CLAUDE.md` with conventions and uv commands.
- `apps/frontend/` — placeholder, not yet scaffolded.
- `infra/` — Docker / deployment scaffolding.
- `docs/superpowers/specs/` — design specs.
- `docs/superpowers/plans/` — implementation plans.

## MCP Rules of Engagement

| MCP | Use when | Don't use when |
|---|---|---|
| `notion-fetch` | User references a backlog story (e.g. "implement 1.3"); user asks to sync backlog status | Session start; questions answerable from this file |
| `notion-search` | Looking up a story by keyword if ID unknown | Anything answerable by `git log` or file reads |
| `context7` | API syntax uncertainty for FastAPI, SQLModel, Alembic, Pydantic, APScheduler | General Python; refactoring; business logic |

The backlog page ID is `357994fa899d808badbee2a9831bc5db`. Cache it; don't re-search.

## Skills (project-scoped, in `.claude/skills/`)

| Skill | Trigger | Purpose |
|---|---|---|
| `mtg-feature-from-backlog` | "implement story X.Y", "from backlog" | Fetches story from Notion, drafts SQLModel + route + migration + test scaffold |
| `scryfall-integration` | working with Scryfall API, card import, bulk data | Reference: endpoints, rate limits, JSON shape, mapping |
| `alembic-migration` | "migration", "alter table", schema change | Strict 5-step recipe with refusal triggers on data loss |
| `mtg-domain-glossary` | MTG-specific terms in user request | Reference: archetypes, formats, mana curve, card types |

## Conventions

- **Package manager:** uv only. Never `pip` or `poetry`.
- **Language:** all code, comments, and Claude tooling files in English. Client-facing artifacts (READMEs in French, with accents) stay in French.
- **No auto-Notion-sync** at session start. Fetch only on explicit reference.
- **Stay in scope.** No drive-by refactors. If a refactor seems necessary, propose it and wait for approval.

## Phase Roadmap

5 epics in the backlog: (1) Collection Management, (2) Friend Groups, (3) Trade System, (4) Deck Building, (5) Smart Suggestions. Phase 1 is the entry point — most current work targets it.
