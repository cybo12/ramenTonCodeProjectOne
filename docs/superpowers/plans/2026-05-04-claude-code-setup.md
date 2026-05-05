# Claude Code Setup for MTG Circle — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Install meta-tooling (CLAUDE.md files + 4 project-scoped skills) so future Claude Code sessions on MTG Circle pick up context with low token cost and predictable conventions.

**Architecture:** Two-tier `CLAUDE.md` (root + per-app) loaded by CWD. Four skills in `.claude/skills/` triggered on demand via the `Skill` tool. MCP usage rules codified in root `CLAUDE.md`. No application code touched.

**Tech Stack:** Markdown only. Verification via `wc -l`, file existence, and YAML frontmatter parsing.

**Spec:** `docs/superpowers/specs/2026-05-04-claude-code-setup-design.md`

---

## File Structure

Files created (8 total):

| Path | Purpose | Target size |
|---|---|---|
| `CLAUDE.md` | Root project context, MCP rules, skill index | ~80 lines |
| `apps/backend/CLAUDE.md` | Backend conventions, uv, FastAPI/SQLModel patterns | ~120 lines |
| `.claude/skills/mtg-feature-from-backlog/SKILL.md` | Workflow: backlog story → 4-part scaffold | ~70 lines |
| `.claude/skills/scryfall-integration/SKILL.md` | Reference: endpoints, rate limits, JSON shape | ~80 lines |
| `.claude/skills/alembic-migration/SKILL.md` | Rigid recipe: 5-step schema change workflow | ~70 lines |
| `.claude/skills/mtg-domain-glossary/SKILL.md` | Reference: archetypes, formats, mana curve | ~70 lines |

No frontend `CLAUDE.md` (deferred — frontend not scaffolded). No `.claude/settings.json` changes.

---

## Task 1: Create directory structure

**Files:**
- Create: `.claude/skills/mtg-feature-from-backlog/` (directory)
- Create: `.claude/skills/scryfall-integration/` (directory)
- Create: `.claude/skills/alembic-migration/` (directory)
- Create: `.claude/skills/mtg-domain-glossary/` (directory)

- [ ] **Step 1: Create the four skill directories**

```bash
mkdir -p .claude/skills/mtg-feature-from-backlog \
         .claude/skills/scryfall-integration \
         .claude/skills/alembic-migration \
         .claude/skills/mtg-domain-glossary
```

- [ ] **Step 2: Verify structure exists**

```bash
ls -la .claude/skills/
```

Expected: 4 directories listed.

---

## Task 2: Write root CLAUDE.md

**Files:**
- Create: `CLAUDE.md`

- [ ] **Step 1: Write the file with this exact content**

````markdown
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
````

- [ ] **Step 2: Verify line count is in target range (~80 lines, hard cap 100)**

```bash
wc -l CLAUDE.md
```

Expected: between 60 and 100 lines.

- [ ] **Step 3: Verify it renders cleanly**

```bash
head -5 CLAUDE.md && echo "---" && tail -5 CLAUDE.md
```

Expected: clean Markdown, no broken tables, no stray characters.

---

## Task 3: Write backend CLAUDE.md

**Files:**
- Create: `apps/backend/CLAUDE.md`

- [ ] **Step 1: Write the file with this exact content**

````markdown
# Backend — apps/backend

FastAPI + SQLModel service on Python 3.14, managed by uv. The root `CLAUDE.md` covers project-wide context and MCP rules; this file covers backend-specific conventions.

## Directory Map

```
app/
├── api/v1/           # routers — register routes here, one file per resource
│   ├── router.py     # aggregates all v1 routers
│   └── routes/       # individual route files (items.py, algorithms.py, ...)
├── core/             # config, constants, logging, security
├── db/
│   ├── base.py       # SQLModel metadata + import all models so Alembic sees them
│   ├── session.py    # engine + get_session dependency
│   └── models/       # one file per table (Card, Item, ExternalSource, ...)
├── services/         # business logic — no FastAPI imports
│   └── external_api/ # Scryfall integration (sync, mapper, client)
├── utils/            # cross-cutting helpers (exceptions, ...)
└── main.py           # FastAPI app factory
```

`alembic/` sits at the package root for migrations. `tests/` mirrors `app/`.

## uv Commands

```bash
uv sync                                          # install/update deps
uv add <package>                                 # add runtime dep
uv add --dev <package>                           # add dev dep
uv run uvicorn app.main:app --reload             # run dev server
uvx pytest                                       # run tests (no install needed)
uvx pytest tests/path/test_x.py -v               # run a single test file
uvx ruff check . --fix                           # lint with autofix
uvx black .                                      # format
uvx mypy app/                                    # type check
uvx alembic revision --autogenerate -m "<msg>"   # create migration
uvx alembic upgrade head                         # apply migrations
```

Never use `pip` or `poetry`. Never activate the `.venv` manually — `uv run` and `uvx` handle that.

## FastAPI Conventions

- **Layering:** `api/v1/routes/<resource>.py` (router) → `services/<domain>.py` (business logic) → `db/models/<resource>.py` (SQLModel) → DB.
- **Routers** declare `APIRouter(prefix=..., tags=[...])` and depend on `get_session` from `db/session.py`.
- **Services** never import from `fastapi`; they take `Session` as a parameter. This keeps them testable and reusable.
- **Schemas** (read/write Pydantic models) live in the same file as the table model — SQLModel encourages co-location.
- **Errors** raise from `utils/exceptions.py`; let FastAPI's exception handlers map to HTTP responses.

## SQLModel Conventions

- One table model per file in `db/models/`. File name is the resource (singular): `card.py`, `item.py`.
- Table model class: `class Card(SQLModel, table=True)`. Set `table=True` only on persistence models.
- Read/write schemas in the same file: `class CardRead(SQLModel)` / `class CardCreate(SQLModel)`, no `table=True`.
- Primary keys: `id: int | None = Field(default=None, primary_key=True)`.
- Always import the new model in `db/base.py` so Alembic autogenerate detects it.

## Alembic Conventions

- Migration messages: imperative, lowercase, short. e.g. `"add card table"`, `"add color_identity to card"`.
- **Always review the autogenerated diff** before `upgrade head`. Watch for unintended `op.drop_*` calls.
- Implement `downgrade()` even for additive migrations.
- Invoke the `alembic-migration` skill on any schema change — it enforces the full 5-step recipe.

## Tests

- Tests live in `tests/`, mirroring `app/`.
- Use `pytest-asyncio` for async tests: decorate with `@pytest.mark.asyncio`.
- Prefer integration tests against a real DB (Docker Compose) over mocks for repository-level code.
- Run a single test: `uvx pytest tests/path/test_x.py::test_name -v`.

## Environment

`.env` keys (see `.env.example` for the template):

- `DATABASE_URL` — PostgreSQL connection string
- `REDIS_URL` — Redis connection string
- `SCRYFALL_BASE_URL` — defaults to `https://api.scryfall.com`
- `LOG_LEVEL` — `INFO` / `DEBUG`

## Skills That Fire In This Directory

- `mtg-feature-from-backlog` — implementing user stories from the Notion backlog.
- `alembic-migration` — any schema change.
- `scryfall-integration` — Scryfall API work.
- `mtg-domain-glossary` — when MTG terms need disambiguation.
````

- [ ] **Step 2: Verify line count**

```bash
wc -l apps/backend/CLAUDE.md
```

Expected: between 90 and 130 lines.

---

## Task 4: Write `mtg-feature-from-backlog` skill

**Files:**
- Create: `.claude/skills/mtg-feature-from-backlog/SKILL.md`

- [ ] **Step 1: Write the file with this exact content**

````markdown
---
name: mtg-feature-from-backlog
description: Use when the user references a backlog story by ID (e.g. "implement 1.3", "let's do story 2.4", "from the backlog") or asks to start work on a Notion-tracked feature. Fetches the story from Notion and drafts a 4-part scaffold (SQLModel + Alembic migration + FastAPI route + pytest test) following existing patterns in apps/backend/.
---

# MTG Feature From Backlog

Workflow for implementing a user story from the Notion backlog without re-deriving project conventions each session.

## When to use

- User says "implement 1.3", "let's do story 2.4", "from the backlog", "user story X.Y".
- User asks to scaffold work tied to one of the 5 epics (Collection, Friend Groups, Trade, Deck Building, Suggestions).

## When NOT to use

- User asks for a code change that isn't backlog-driven (e.g. "fix this typo", "add logging here").
- User wants to refactor existing code.

## Procedure

### 1. Fetch the story

The MTG Circle backlog page ID is `357994fa899d808badbee2a9831bc5db`. Fetch it via:

```
mcp__notion__notion-fetch with id="357994fa899d808badbee2a9831bc5db"
```

If the page is unreachable or the ID has rotated, fall back to `mcp__notion__notion-search` with `query="MTG Circle Backlog"` and `query_type="internal"`.

### 2. Identify the story row

Find the row where the `#` column matches the user's reference (e.g. `1.3`). Extract:

- The User Story text (acceptance hint).
- The epic (from the section heading above the table).
- The priority (`🔴 Must` / `🟡 Should` / `🟢 Nice`).

### 3. Read existing patterns

Before drafting, read the closest equivalent files in `apps/backend/app/` so the new code matches the project's style. Always consult:

- `app/db/models/item.py` for table-model shape.
- `app/api/v1/routes/items.py` for router shape.
- `app/db/base.py` to confirm where to register the new model.
- The existing `tests/` directory for the test layout.

### 4. Draft the 4-part scaffold

Propose to the user (do not write yet):

1. **SQLModel** in `app/db/models/<resource>.py` — table model + Read/Create schemas. Co-located.
2. **Alembic migration** — describe the autogenerate command and what columns the diff should contain. Defer the actual migration to the `alembic-migration` skill.
3. **FastAPI router** in `app/api/v1/routes/<resource>.py` — only the endpoints implied by the story's acceptance hint (YAGNI). Wire it into `app/api/v1/router.py`.
4. **Pytest test** in `tests/api/v1/test_<resource>.py` — at least one happy-path integration test using `pytest-asyncio` against the DB.

### 5. Hand off for approval

Present the scaffold as a numbered list with the file paths and the rationale for each piece. Wait for the user to approve before writing files. Do NOT write code in this skill — the skill stops at the scaffold proposal.

## Anti-patterns to avoid

- Inventing a new layering (e.g. putting business logic in routes). Match existing patterns.
- Adding helpers / abstractions not implied by the story. YAGNI.
- Skipping the Notion fetch and assuming the story content from memory — backlogs change.
- Writing the migration directly. Hand off to `alembic-migration`.
````

- [ ] **Step 2: Verify the YAML frontmatter parses**

```bash
head -4 .claude/skills/mtg-feature-from-backlog/SKILL.md
```

Expected: lines 1 and 4 are `---`; line 2 starts with `name:`; line 3 starts with `description:`.

- [ ] **Step 3: Verify line count**

```bash
wc -l .claude/skills/mtg-feature-from-backlog/SKILL.md
```

Expected: between 50 and 90 lines.

---

## Task 5: Write `scryfall-integration` skill

**Files:**
- Create: `.claude/skills/scryfall-integration/SKILL.md`

- [ ] **Step 1: Write the file with this exact content**

````markdown
---
name: scryfall-integration
description: Use when working with the Scryfall API — fetching card data, importing collections, bulk data downloads, card search by name/oracle/set, or mapping Scryfall JSON to the project's Card model. Reference for endpoints, rate limits, JSON shape, and bulk-vs-search trade-offs.
---

# Scryfall Integration

Reference for the Scryfall API (https://scryfall.com/docs/api), the canonical free MTG card database.

## Base URL

`https://api.scryfall.com` — no auth required for public endpoints.

## Rate Limits

- **50–100 ms minimum between requests.** Aim for 10 req/s max.
- The API enforces this; bursting causes 429 with a Retry-After header.
- Add `time.sleep(0.1)` between calls in tight loops, or use a token bucket.
- For full-collection sync, prefer bulk data — it's a single download.

## Key Endpoints

| Endpoint | Purpose |
|---|---|
| `GET /cards/named?exact=Lightning+Bolt` | Single card by exact name |
| `GET /cards/named?fuzzy=lightnig+bolt` | Single card with fuzzy match (typo-tolerant) |
| `GET /cards/search?q=t:creature+c:r` | Search with Scryfall query syntax |
| `GET /cards/{id}` | Single card by Scryfall UUID |
| `GET /sets` | All sets/expansions |
| `GET /bulk-data` | List of bulk JSON downloads (default cards, all cards, etc.) |

## Bulk vs Search Decision

- **Bulk data** (`/bulk-data` → download `default_cards` JSON, ~500 MB): use for initial DB seeding, periodic full refresh, anything needing >1000 cards. Updated daily ~05:00 UTC.
- **Search endpoint**: use for user-driven lookups, autocomplete, single-card queries. Paginate via `next_page` field.
- **Named endpoint**: use when importing from a CSV / text decklist where each line is `1 Lightning Bolt`.

Rule of thumb: if you'd hit the API more than 100 times for one task, switch to bulk.

## Card JSON Shape (relevant fields)

```json
{
  "id": "uuid",
  "oracle_id": "uuid",
  "name": "Lightning Bolt",
  "mana_cost": "{R}",
  "cmc": 1.0,
  "type_line": "Instant",
  "oracle_text": "Lightning Bolt deals 3 damage to any target.",
  "colors": ["R"],
  "color_identity": ["R"],
  "set": "lea",
  "set_name": "Limited Edition Alpha",
  "rarity": "common",
  "image_uris": { "normal": "https://..." },
  "prices": { "eur": "2.50", "usd": "3.20" }
}
```

## Mapping to Our `Card` Model

- `id` (Scryfall) → `scryfall_id` (string).
- `oracle_id` → `oracle_id` (string, indexed) — this is the "logical card" key. Two reprints of Lightning Bolt share the same `oracle_id`.
- `colors` and `color_identity` are arrays of `W/U/B/R/G` — store as `JSON` or normalize to a many-to-many.
- `prices.eur` / `prices.usd` are strings or null — convert with care, may be `None` for un-priced cards.

## Caching Guidance

Scryfall explicitly recommends client-side caching.

- Cache bulk-data downloads on disk; check the `updated_at` timestamp before re-downloading.
- For dev search/named queries, an in-memory dict keyed by query string is enough.
- Don't re-fetch the same card by ID within one request lifecycle.

## Common Pitfalls

- **Double-faced cards** (DFCs) have a `card_faces` array; `mana_cost` and `oracle_text` may be empty at the root. Read `card_faces[0]` and `card_faces[1]`.
- **Foreign-language printings** are separate objects. Filter `lang == "en"` unless you specifically want others.
- **Tokens and emblems** appear in bulk data with `layout: "token"` / `"emblem"` — exclude when building player collections.
````

- [ ] **Step 2: Verify frontmatter and line count**

```bash
head -4 .claude/skills/scryfall-integration/SKILL.md && wc -l .claude/skills/scryfall-integration/SKILL.md
```

Expected: valid frontmatter; 60–100 lines.

---

## Task 6: Write `alembic-migration` skill

**Files:**
- Create: `.claude/skills/alembic-migration/SKILL.md`

- [ ] **Step 1: Write the file with this exact content**

````markdown
---
name: alembic-migration
description: Use whenever a schema change is needed — adding/removing/renaming columns, creating tables, altering types, or any modification to a SQLModel marked table=True in apps/backend/app/db/models/. Enforces a strict 5-step recipe with refusal triggers on data-loss diffs.
---

# Alembic Migration Recipe

Rigid workflow. Follow the steps in order. Do not skip step 3.

## When to use

- User asks to "add a migration", "alter table", "add a column".
- A SQLModel in `app/db/models/` was modified (field added/removed/renamed/retyped).
- A new table model was added.

## When NOT to use

- The change is to a non-`table=True` schema (Read/Create models). No migration needed.
- The change is in `services/` or `api/` only.

## The 5 Steps

### Step 1 — Edit the SQLModel

Edit the file in `app/db/models/<resource>.py`. If you added a new model file, also import it in `app/db/base.py`:

```python
# app/db/base.py
from app.db.models.card import Card  # noqa: F401
```

### Step 2 — Autogenerate the migration

From `apps/backend/`:

```bash
uvx alembic revision --autogenerate -m "<imperative lowercase description>"
```

Examples: `"add card table"`, `"add color_identity to card"`, `"rename item.value to item.amount"`.

### Step 3 — Review the diff (CRITICAL)

Open `alembic/versions/<rev>_<msg>.py`. Read both `upgrade()` and `downgrade()`.

**Refuse to proceed if any of these appear without explicit user opt-in:**

- `op.drop_table(...)` on an existing table that has data.
- `op.drop_column(...)` on a column the user didn't explicitly ask to remove.
- `op.alter_column(..., nullable=False)` on a column that may have existing nulls (will fail at runtime).
- `op.alter_column(..., type_=...)` that narrows the type (e.g. `String(255)` → `String(50)`).
- Empty `downgrade()` body (Alembic generated `pass` because the change is one-way).

If you see any of these, stop and ask the user. State which line is the concern.

### Step 4 — Apply the migration

```bash
uvx alembic upgrade head
```

If it fails, do not auto-retry. Read the error, fix the model or migration, and re-run autogenerate or edit the migration by hand.

### Step 5 — Verify

Pick one:

- Open Adminer at `http://localhost:8080`, log in to the `db` service, inspect the table schema.
- Or: `docker compose exec db psql -U <user> -d <db> -c "\d <table_name>"`.

Confirm the column / table matches the model. Then commit:

```bash
git add app/db/models/<resource>.py app/db/base.py alembic/versions/<rev>_<msg>.py
git commit -m "feat(db): <description>"
```

## Anti-patterns

- Editing the migration file by hand to silence a confusing diff. Fix the model instead and re-autogenerate.
- Running `upgrade head` before reading the diff.
- Marking a column not-null without a server default on a table with existing rows.
- Skipping `downgrade()` — even additive migrations should be reversible.
````

- [ ] **Step 2: Verify frontmatter and line count**

```bash
head -4 .claude/skills/alembic-migration/SKILL.md && wc -l .claude/skills/alembic-migration/SKILL.md
```

Expected: valid frontmatter; 50–90 lines.

---

## Task 7: Write `mtg-domain-glossary` skill

**Files:**
- Create: `.claude/skills/mtg-domain-glossary/SKILL.md`

- [ ] **Step 1: Write the file with this exact content**

````markdown
---
name: mtg-domain-glossary
description: Use when the user message contains MTG-specific terminology that affects design — CMC, mana curve, color identity, archetype names (Aggro/Control/Combo/Midrange/Mill/Tokens/Burn/Ramp), format names (Foundation/Standard/Modern/Commander), card types, or trade vocabulary. Reference for accurate domain modeling without asking the user to define terms.
---

# MTG Domain Glossary

Reference for Magic: The Gathering vocabulary used throughout MTG Circle.

## Card Mechanics

- **CMC (Converted Mana Cost) / Mana Value** — total mana cost of a card. `{2}{R}{R}` → CMC 4. Modern term: "mana value".
- **Mana cost** — the per-color breakdown. `{2}{R}{R}` means 2 generic + 2 red.
- **Color identity** — every colored mana symbol on a card (cost + rules text). Used for Commander deck legality. A card can have a colorless cast cost but a non-empty color identity.
- **Colors** — only colors in the cast cost (subset of color identity).
- **Card types** — Creature, Instant, Sorcery, Artifact, Enchantment, Planeswalker, Land, Battle, Tribal.
- **Supertypes** — Legendary, Basic, Snow, World.
- **Subtypes** — creature types (Goblin, Wizard…), spell types (Arcane), land types (Mountain, Plains…).

## Mana Curve

The distribution of CMCs in a deck, usually shown as a bar chart (CMC 0, 1, 2, 3, 4, 5, 6+).

- **Low curve** (Aggro): peak at CMC 1–2, few cards above 4.
- **Mid curve** (Midrange): peak at CMC 2–4.
- **High curve** (Control / Ramp): meaningful presence at CMC 5+, supported by ramp or card advantage.

A "good curve" is archetype-dependent — there is no universal target. The MTG Circle deck builder should let the user *see* their curve, not impose a target.

## Archetypes (referenced in backlog story 1.4)

- **Aggro** — fast creatures, low curve, win by turn 4–5. Goal: deal 20 damage quickly.
- **Control** — counterspells, removal, card draw. Win late with a single big threat.
- **Combo** — assemble specific card interactions for an instant win (e.g. infinite mana → infinite damage).
- **Midrange** — flexible threats and answers, adapts to opponent. CMC 2–4 sweet spot.
- **Mill** — win by emptying the opponent's library, not their life total.
- **Tokens** — generate many small creatures; win through sheer board presence.
- **Burn** — direct damage spells (Lightning Bolt, Fireball) targeting the opponent.
- **Ramp** — accelerate mana production to cast big spells early.

## Formats

| Format | Deck size | Singleton | Card pool |
|---|---|---|---|
| Foundation | 60+ | No (4-of) | Foundations + Standard-legal sets |
| Standard | 60+ | No | ~last 2 years of sets |
| Pioneer | 60+ | No | Return to Ravnica (2012) onward |
| Modern | 60+ | No | 8th Edition (2003) onward |
| Legacy | 60+ | No | All sets, banned list applies |
| Vintage | 60+ | No | All sets, restricted list (1-of for some cards) |
| Commander / EDH | 100 | Yes (1-of) | Identity must match commander; multiplayer |

The MTG Circle backlog targets MTG **Foundation** as the primary format, but the data model should not hard-code a format — store the legal formats as metadata on each card.

## Trade Vocabulary

- **Open to trade** — user has marked a card as available for proposals.
- **Wishlist** — cards a user wants. Can be public (signal to friends) or private.
- **Watchlist** — cards a user is monitoring (price, availability) without intent to acquire immediately.
- **Value-equivalent / equitable trade** — cards exchanged with similar market value. The "trade balance score" in EPIC 3 (story 3.2/3.3) compares CardMarket prices.
- **Foil / non-foil** — same card, different finish. Different prices. The model should track this as a separate variant.
- **Condition** — Mint / Near Mint / Excellent / Good / Poor. Affects value heavily.
````

- [ ] **Step 2: Verify frontmatter and line count**

```bash
head -4 .claude/skills/mtg-domain-glossary/SKILL.md && wc -l .claude/skills/mtg-domain-glossary/SKILL.md
```

Expected: valid frontmatter; 50–90 lines.

---

## Task 8: Final verification

- [ ] **Step 1: Confirm all 6 files exist with expected sizes**

```bash
wc -l CLAUDE.md \
      apps/backend/CLAUDE.md \
      .claude/skills/mtg-feature-from-backlog/SKILL.md \
      .claude/skills/scryfall-integration/SKILL.md \
      .claude/skills/alembic-migration/SKILL.md \
      .claude/skills/mtg-domain-glossary/SKILL.md
```

Expected: 6 lines of output, each between 50 and 130 lines, no "No such file" errors.

- [ ] **Step 2: Confirm skill frontmatter is valid on all four**

```bash
for f in .claude/skills/*/SKILL.md; do
  echo "=== $f ==="
  head -5 "$f"
done
```

Expected: each file starts with `---`, has `name:` and `description:` fields, ends frontmatter with `---` on line 4.

- [ ] **Step 3: Confirm directory tree**

```bash
ls -la .claude/skills/ && tree .claude/skills/ 2>/dev/null || find .claude/skills/ -type f
```

Expected: 4 skill directories, each containing exactly one `SKILL.md`.

---

## Task 9: Commit

- [ ] **Step 1: Stage and commit**

```bash
git add CLAUDE.md apps/backend/CLAUDE.md .claude/skills/ docs/superpowers/plans/2026-05-04-claude-code-setup.md
git status
```

Expected: only the 7 new files staged (no stray modifications from elsewhere). If `git status` shows other modified files staged, unstage them with `git restore --staged <file>` before committing.

- [ ] **Step 2: Create the commit**

```bash
git commit -m "$(cat <<'EOF'
chore: add Claude Code setup (CLAUDE.md + project skills)

Adds root and backend CLAUDE.md with stack, conventions, and MCP rules of
engagement. Adds four project-scoped skills (mtg-feature-from-backlog,
scryfall-integration, alembic-migration, mtg-domain-glossary) under
.claude/skills/ to encode common workflows for MTG Circle development.
No application code touched.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
EOF
)"
```

- [ ] **Step 3: Verify the commit**

```bash
git show --stat HEAD
```

Expected: 7 files added, ~500-600 insertions, no deletions, no unrelated files.
