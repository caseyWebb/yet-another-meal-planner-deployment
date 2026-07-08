---
name: shop-groceries
description: "Flush the grocery list — the deliberate act distinct from capturing intent. Use for \"place the order\", \"I'm headed to the store\", \"give me a shopping list\", \"I'm walking Central Market\", \"send it to my cart\", \"go ahead and order\". Detects the fulfillment mode and runs the right branch: Kroger online cart flush, Kroger in-store API-ordered walk, mapped-store walk, or map-and-walk. The only path that writes the cart or transitions list items to received."
---

> **Prerequisite** — if you haven't already this session, read the `grocery-core` and `grocery-cart` skills before continuing.

# Shop groceries — the flush (shop-groceries)

Read `read_to_buy` and `read_user_profile()` in parallel — `read_to_buy` is the shop-time read (the active list ∪ the meal plan's derived needs − pantry on-hand, the same set every flush resolves; `read_grocery_list` shows only the stored rows and would miss the plan); the profile's preferences field drives branch detection. Surface `underived` up front in any branch — those planned recipes' items are NOT in the set. Then detect which branch to run:

| Signal | Branch |
|---|---|
| `primary = "kroger"` and no store named for this trip | **Kroger online** — `place_order` flush |
| `primary = "kroger"` and I named a specific Kroger store, or I say "in-store" / "walking the Kroger" | **Kroger in-store** — API aisle ordering |
| `primary` is a store slug marked `fulfillment: "satellite"` (from `read_user_profile()`) | **Satellite cart-fill** — point me at my local cart-fill helper; no `place_order`, no walk list |
| `primary` is a store slug (not satellite-marked), or I named a non-Kroger store | **In-store walk** — layout/notes aisle ordering |
| Walking a store we've never mapped and I want to record it | **Map + walk** — concurrent map-and-shop |

> For details, read `references/kroger-online.md`.

> For details, read `references/kroger-instore.md`.

> For details, read `references/satellite-cartfill.md`.

> For details, read `references/instore-walk.md`.

> For details, read `references/map-store.md`.
