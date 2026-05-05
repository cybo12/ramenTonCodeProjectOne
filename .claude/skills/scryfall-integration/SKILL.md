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
