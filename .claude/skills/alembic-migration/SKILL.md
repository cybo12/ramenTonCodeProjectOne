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
