# Claude Code Setup for MTG Circle — Design

**Date:** 2026-05-04
**Author:** Claude (under direction of cybo12)
**Status:** Approved (brainstorming phase) — pending user review of written spec

## 1. Context

MTG Circle is a web platform (envisioned stack: Next.js + Tailwind, FastAPI + SQLModel, PostgreSQL) that lets a group of friends manage their Magic: The Gathering collections, propose trades, and build decks intelligently. The product backlog lives in Notion under "🃏 MTG Circle — Backlog" (5 epics: Collection, Friend Groups, Trade, Deck Building, Smart Suggestions). The backend already exists with uv + FastAPI + SQLModel + Alembic + PostgreSQL on Python 3.14, plus an embryonic `external_api/` service for Scryfall integration.

The user wants to resume work on this project with Claude Code as the primary assistant. The goal of this spec is to install the *meta-tooling* — `CLAUDE.md` files and project-scoped skills — that makes future sessions efficient (low token usage, predictable conventions, smart MCP usage). No application code is touched in this spec.

## 2. Goals

- Future Claude Code sessions pick up project context with zero hand-holding.
- Token budget per session is minimized: only the relevant `CLAUDE.md` is loaded based on CWD; skills are triggered on demand, not preloaded.
- MCP servers (Notion, context7) are used surgically, with codified rules.
- Common workflows for this project (backlog-driven feature work, Scryfall integration, Alembic migrations, MTG domain reasoning) are encoded as skills so the assistant doesn't re-derive them each session.

## 3. Non-Goals

- No refactor of existing `apps/backend/app/services/external_api/` code (deferred to a follow-up session).
- No CardMarket skill (deferred — Phase 3 not started).
- No frontend `CLAUDE.md` content (placeholder only — frontend doesn't exist yet).
- No `.claude/settings.json` permission tuning (separate, on-demand task).
- No application-level documentation rewrite (READMEs stay as-is).

## 4. File Layout

```
/CLAUDE.md                                         # root: project vision, stack, MCP rules, skill index
/apps/backend/CLAUDE.md                            # backend: uv, FastAPI/SQLModel conventions
/apps/frontend/CLAUDE.md                           # placeholder (created when frontend lands)
/.claude/skills/mtg-feature-from-backlog/SKILL.md
/.claude/skills/scryfall-integration/SKILL.md
/.claude/skills/alembic-migration/SKILL.md
/.claude/skills/mtg-domain-glossary/SKILL.md
/docs/superpowers/specs/2026-05-04-claude-code-setup-design.md   # this file
```

Project-scoped skills live under `.claude/skills/` so they are versioned with the repo and shared with teammates who clone it.

The frontend `CLAUDE.md` is intentionally deferred. Creating it empty would just clutter; it will be added when the frontend app is scaffolded.

## 5. Language Policy

- All `CLAUDE.md` files and skill content: **English** (industry standard, denser tokens, easier interop with model defaults).
- Client-facing artifacts (READMEs, French guides, Notion content): **French with accents** (consistent with existing project tone).
- This spec doc: English (it's tooling documentation, not client-facing).

## 6. CLAUDE.md Sizing & Content

### 6.1 Root `CLAUDE.md` (~80 lines target)

Sections:
- **One-line vision** (MTG Circle pitch).
- **Tech stack** as a table (backend / frontend / infra).
- **Monorepo layout** (3 lines: `apps/backend`, `apps/frontend` (planned), `infra`).
- **MCP rules of engagement** (table: Notion → only on backlog references; context7 → only on API uncertainty; never auto-fetch on session start).
- **Skill index** (4 rows: name, trigger, one-line purpose).
- **Do-not list** (no `pip`/`poetry` — uv only; no English in client-facing content; no auto-Notion-sync).

### 6.2 Backend `CLAUDE.md` (~120 lines target)

Sections:
- **Stack reminder** (1 line).
- **Directory map** of `app/` (api, core, db, services, utils — 6 lines).
- **uv command cheatsheet** (sync, add, run, exec — copy-paste-ready).
- **FastAPI conventions**: route → service → repository → model layering; where validators live; how dependency injection is wired.
- **SQLModel conventions**: table model vs schema model; relationships; primary key style.
- **Alembic conventions**: naming, autogenerate flow, review-diff requirement.
- **Test layout**: where pytest finds tests, async test pattern.
- **Env keys** (one-line list of `.env` variable names — values redacted).
- **Pointer to skills** (one line each, with trigger).

## 7. MCP Rules of Engagement

Codified in root `CLAUDE.md`:

| MCP | Use when | Don't use when |
|---|---|---|
| `notion-fetch` | User references a backlog story (e.g. "1.3"); user asks to sync backlog status | Session start; general project questions answerable from `CLAUDE.md` |
| `notion-search` | Looking up a story by keyword if ID unknown | Anything answerable by `git log` / file reads |
| `context7` | API syntax uncertainty for FastAPI, SQLModel, Alembic, Pydantic, APScheduler | General Python questions; refactoring; business logic |

## 8. Skills

All four skills are project-scoped (`.claude/skills/<name>/SKILL.md`), each with YAML frontmatter (`name`, `description`, optional `when_to_use`). The `description` is critical — it determines whether Claude auto-invokes the skill.

### 8.1 `mtg-feature-from-backlog` (flexible workflow)

**Trigger:** "implement story X.Y", "from backlog", "user story X", or any direct reference to a Notion-tracked feature.

**Behavior:**
1. Fetch the story from Notion via `mcp__notion__notion-fetch` (use the cached page ID where possible).
2. Identify epic and acceptance hints from the story row.
3. Propose a 4-part scaffold:
   - SQLModel (table + read/write schemas) in `app/db/models/`.
   - Alembic migration (autogenerate, then review diff).
   - FastAPI router + service in `app/api/v1/routes/` and `app/services/`.
   - Pytest async test in `tests/`.
4. Follow existing patterns observed in `app/` — never invent a new layering.
5. Hand off to the user for approval before writing code.

### 8.2 `scryfall-integration` (reference)

**Trigger:** working with Scryfall API, card import, bulk data, card search.

**Content:**
- Base URL, key endpoints (`/cards/named`, `/cards/search`, `/bulk-data`).
- Rate limit guidance: 50-100 ms between requests, max 10 req/s.
- Bulk-data vs search: when to download bulk JSON (full collection sync) vs hit the search endpoint (single-card lookup).
- JSON shape: critical fields (`id`, `oracle_id`, `name`, `mana_cost`, `cmc`, `type_line`, `colors`, `color_identity`, `set`, `rarity`, `prices`).
- Mapping rules to our `Card` model.
- Caching guidance (Scryfall recommends client-side cache).
- Pointer to official docs URL.

### 8.3 `alembic-migration` (rigid recipe)

**Trigger:** "migration", "alter table", schema change, model field added/removed.

**5-step recipe (strict order):**
1. Edit / add the SQLModel in `app/db/models/`.
2. Run `uvx alembic revision --autogenerate -m "<imperative description>"`.
3. **Review the generated diff** — refuse to proceed if it includes unintended `op.drop_*` on existing tables/columns or could lose data.
4. Run `uvx alembic upgrade head`.
5. Verify in Adminer (`http://localhost:8080`) or `psql` that the schema reflects expectation.

**Refusal triggers:** any data-loss diff without explicit user opt-in; missing rollback (`downgrade()`) implementation.

### 8.4 `mtg-domain-glossary` (reference)

**Trigger:** user message contains MTG-specific terms (CMC, mana curve, color identity, archetype names, format names, card types).

**Content:**
- **Card mechanics**: CMC, mana cost, color identity, types (Creature, Instant, Sorcery, Artifact, Enchantment, Planeswalker, Land, Battle).
- **Archetypes** (referenced explicitly in backlog 1.4): Aggro, Control, Combo, Midrange, Mill, Tokens, Burn, Ramp.
- **Mana curve**: definition, why it matters, what "good curve" means per archetype.
- **Formats**: Foundation, Standard, Pioneer, Modern, Legacy, Vintage, Commander/EDH (deck-size and singleton implications).
- **Trade vocabulary**: open-to-trade, wishlist, watchlist, value-equivalent.

## 9. Token Optimization Levers

- Backend `CLAUDE.md` is only loaded when CWD is `apps/backend/` or below — frontend/infra sessions don't pay its cost.
- Skills are loaded on demand via the `Skill` tool — their bodies don't sit in context unless triggered.
- MCP rules forbid auto-fetch on session start.
- `mtg-feature-from-backlog` caches the Notion page ID in the skill body so the assistant doesn't re-search.
- All skill bodies stay under ~150 lines; reference content uses tables / bullet lists, not prose.

## 10. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Skill `description` field too vague → not auto-invoked | Each skill has explicit trigger phrases listed in description |
| `CLAUDE.md` drift from actual code | Skills point to *patterns*, code is the source of truth; CLAUDE.md is updated when conventions change |
| Notion backlog re-organized → cached page ID stale | `mtg-feature-from-backlog` falls back to `notion-search` if fetch fails |
| Skills become obsolete (e.g. CardMarket added later) | Skills versioned with repo; deletion is a normal PR |

## 11. Validation

After implementation, validate by:
1. `ls .claude/skills/` shows 4 directories.
2. `cat CLAUDE.md` and `cat apps/backend/CLAUDE.md` render cleanly and stay under target line counts.
3. Open a fresh Claude Code session and ask "what's the stack?" — answer should come without tool calls.
4. Ask "implement story 1.3" — assistant should invoke `mtg-feature-from-backlog`, fetch Notion, propose scaffold.

## 12. Out of Scope (Tracked for Later)

- Refactor / rewrite of `apps/backend/app/services/external_api/` (chosen by user as separate work).
- CardMarket integration skill.
- Frontend `CLAUDE.md` content.
- `.claude/settings.json` permission tuning (use `/fewer-permission-prompts` later).
- Hooks for automated behaviors (none requested).
