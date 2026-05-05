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
