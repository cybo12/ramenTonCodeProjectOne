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
