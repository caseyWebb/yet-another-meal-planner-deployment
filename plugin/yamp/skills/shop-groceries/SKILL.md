---
name: shop-groceries
description: "Flush the grocery list — the deliberate act distinct from capturing intent. Use for \"place the order\", \"I'm headed to the store\", \"give me a shopping list\", \"I'm walking Central Market\", \"shop on Instacart\", \"send it to my cart\", \"go ahead and order\". Detects explicit Instacart trip intent or the configured fulfillment mode and runs the right handoff, cart, or walk branch."
---

> **Prerequisite** — if you haven't already this session, read the `yamp-core` and `yamp-cart` skills before continuing.

# Shop groceries — the flush (shop-groceries)

Read `read_to_buy({ enrich: true })` and `read_user_profile()` in parallel — `read_to_buy` is the shop-time read (the active list ∪ the meal plan's derived needs − pantry on-hand, the same set every flush resolves; `read_grocery_list` shows only the stored rows and would miss the plan); the profile's preferences field drives branch detection. `enrich` costs at most one Kroger Locations resolve and zero product searches, and pays for two things on every line at once: aisle `placement` where a store resolves, and `substitutes[]` — relation-labeled cross-ingredient siblings from the identity graph, each flagged `in_pantry` and/or `on_sale_hint` — plus `flyer_as_of` on the view. This runs the same way in **every** branch, walk and satellite included: with no resolvable Kroger location the read still serves `in_pantry` hits and label-keyed `on_sale_hint`s at zero Kroger cost, just without aisle placement.

Surface `underived` up front in any branch — those planned recipes' items are NOT in the set. **Walk the substitute hints here too, at list review, before any branch-specific work:** a sibling flagged `in_pantry` means I may already have a stand-in and might not need to buy the line at all; an `on_sale_hint` sibling names a substitute that's on sale — cite its real price, not a bare claim. The tool proposes and *names* each relation — whether it actually fits the dish is **yours** to judge, grounded in its data; skip weak ones rather than reciting them. What I accept maps onto the existing writes right away: on an **explicit** row, `add_to_grocery_list` (note the swap) + `remove_from_grocery_list`; on a **plan-derived** row, materialize the sibling with `add_to_grocery_list` now, and — for a Kroger-online flush — stage an order-scoped `exclude` of the original to carry onto `place_order` at step 5 (the plan still lists it; nothing is suppressed until the flush runs). A branch with no cart flush (walk, in-store, satellite) has nothing to stage — just don't pick up the original at the shelf, same as any planned item I decide to skip this trip. What I decline, drop without comment.

**One ready-to-eat offer, here for every branch.** Ask whether I want a heat-and-eat / grab-and-go option and wait for explicit confirmation before adding anything. With a configured catalog, make the offer specific by suggesting a low/out favorite; for a Kroger trip, an on-sale discovery may also make it specific. With an empty catalog, make a generic offer instead. Only on my explicit yes, add the chosen item to the grocery list (or `stockup.toml` for a conditional bulk buy). Do not prompt again inside the selected branch.

Then detect which branch to run:

| Signal | Branch |
|---|---|
| I explicitly ask to use Instacart for this trip | **Instacart Marketplace handoff** — `create_instacart_handoff`; this explicit signal wins over the standing primary store |
| `primary = "kroger"` and no store named for this trip | **Kroger online** — `place_order` flush |
| `primary = "kroger"` and I named a specific Kroger store, or I say "in-store" / "walking the Kroger" | **Kroger in-store** — API aisle ordering |
| `primary` is a store slug marked `fulfillment: "satellite"` (from `read_user_profile()`) | **Satellite cart-fill** — point me at my local cart-fill helper; no `place_order`, no walk list |
| `primary` is an Offline store slug (not satellite-marked), or I named an Offline store | **In-store walk** — layout/notes aisle ordering |
| Walking a store we've never mapped and I want to record it | **Map + walk** — concurrent map-and-shop |

> For details, read `references/instacart-marketplace.md`.

> For details, read `references/kroger-online.md`.

> For details, read `references/kroger-instore.md`.

> For details, read `references/satellite-cartfill.md`.

> For details, read `references/instore-walk.md`.

> For details, read `references/map-store.md`.
