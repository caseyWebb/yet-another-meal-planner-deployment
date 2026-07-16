---
name: shop
description: "Flush the list — order online, walk a store, or hand off. Use for \"place the order\", \"I'm headed to the store\", \"give me a shopping list\", \"shop it on Instacart\", \"I'm at Central Market\". Capture is continuous; this flow is the deliberate flush, branch picked from the household's setup and this trip's words."
---

> **Prerequisite** — if you haven't already this session, read the `yamp-core` skill before continuing.

# Shop (shop)

Read `read_to_buy({ enrich: true })` and `read_user_profile()` in parallel. Surface `underived` up front (those recipes' items are NOT in the set), and walk the substitute hints once at review: a sibling already in the pantry may erase a line; an on-sale sibling is only worth naming with its real price. What I accept becomes list ops now; what I decline, drop silently.

Pick the branch — explicit words for this trip beat standing preference:

| Signal | Branch |
|---|---|
| I explicitly ask for Instacart | Instacart handoff |
| Kroger primary, nothing named | Kroger online order |
| Kroger, but a store named or "in-store" | Kroger in-store walk |
| Primary is a satellite-fulfilled store | Local cart-fill helper |
| Primary or named store is offline | Store walk from layout notes |
| Unmapped store and I'm game | Map as we shop: `add_store`, then `add_store_note` layout lines aisle by aisle |

Check-off, "picked up", and receiving are mine to assert on the list card; a receive restocks the pantry (`update_pantry`) and may earn one or two vetted storage tips (`read_guidance("ingredient_storage")`) for what actually needs them.

> For details, read `references/kroger-online.md`.

> For details, read `references/store-walk.md`.

> For details, read `references/instacart.md`.
