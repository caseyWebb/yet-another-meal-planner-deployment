# Kroger In-Store — API aisle ordering

This branch runs when I'm walking a Kroger store in person. The Kroger Products API returns `aisleLocation` for each item, so the walk is ordered by aisle number automatically — no pre-mapped layout required.

#### 1. Resolve the Kroger store

Check whether a Kroger store is registered for this trip:
- `list_stores()` and look for a store with `chain: "kroger"` matching the location I named or my `primary` preference.
- If found, use its `slug` and read its `location_id` (the Kroger `locationId` used to bypass the Locations API).
- **If not registered (first visit to this Kroger):** ask me for a short label — "What do you want to call this Kroger? (e.g. 'West 7th', 'Hulen')" — then:
  - Derive a kebab slug: `kroger-<label-in-kebab>` (e.g. `kroger-west-7th`).
  - Call `kroger_prices` on any one list item with the store ZIP/label to resolve the Kroger `locationId`.
  - `add_store(slug, name="Kroger", label, chain="kroger", domain="grocery", location_id=<resolved>)`.
  - This is **one-time friction** — subsequent walks resolve by slug with no API lookup needed.

#### 2. Load items and their placements

Open with `read_to_buy({ enrich: true })`: each line may carry a `placement` — a **captured aisle** (`aisle_number`/`aisle_description`, learned from past orders' resolved products and stored on the SKU cache at this store) and/or a graph-derived `department` — and its `substitutes[]`, re-resolved for *this* store if it isn't the one the opening read used. Captured placements are **real store data — prefer them** over anything inferred; they cost no product lookups and cover more of the list with every order placed. (The top-level `location` names the store the placements are for — if it isn't this trip's store, treat the placements as absent.)

For the lines **without** a captured placement, fall back to `kroger_prices` in parallel (plan-derived lines walk exactly like explicit rows, each carrying its `for_recipes` attribution), passing `location_id` (the store's registered Kroger `locationId`; omit to fall back to the profile preferred location). Each returned product carries `aisleLocation: { number, description, side? } | null` and a top-level `inStore: boolean`.

Surface **`inStore: false` items up front** before starting the walk: "These items aren't available in-store at this Kroger — pickup/delivery only. Remove them from the in-store list, or keep them for a separate order?" Never silently drop them.

#### 3. Group by aisle and walk

Order items by aisle number (ascending) — captured placements and `kroger_prices` aisles interleave into one walk; a store note's `location` pin still **wins** over either for its item. Items with no aisle from any source go at the end as **"location unknown"** (grouped by department when the placement carries one). Apply cold-chain sequencing on top: if frozen/refrigerated aisles fall mid-store, pull those items into a final "grab these on your way out" group and say so.

Hands-free / voice-first, **one aisle at a time**, I advance with "got it" / "next". At each aisle, announce the aisle number and description, then the items to grab there. As we reach an aisle, if something there has **purchasing** guidance (which canned tomatoes, which olive oil), weave the non-obvious tip in following the **Picking what to buy** guidance — at the shelf, where I'm choosing.

Handle **"can't find it"** by disambiguating gently before any write:
- **Sold out** — transient, no note.
- **Moved** (I found it in a different aisle) — silently write a corrected `location` note (see §4 below).
- **Not carried** — *offer* a `stock` note (`add_store_note` with `tags:["stock"]`) and note it for the trip.

#### 4. Silent idempotent location note seeding

After `read_store_notes(slug)`, for each item resolved to a specific aisle, silently write:

```
add_store_note(slug, "Aisle <N>: <item name>", tags: ["location"])
```

**Only if** no existing `location`-tagged note already mentions the item name (case-insensitive substring match). This runs silently — no confirmation prompt, no narration. The notes accumulate across trips, so the walk gets faster over time even without a full pre-map.

#### 5. Complete → received

Before wrapping up, sweep the list for anything we never ticked off — "you've still got harissa and flour on the list; did we pass those, or want to double back?" Then, when done, picked items go straight `active → received` — **no `in_cart`/`ordered` stage**. Persist it with the granular tools: remove the picked **explicit rows** with `remove_from_grocery_list` (one per item, awaited) and — **for `grocery`-kind items only** — restock the pantry in one `update_pantry({ operations: [...] })`; `household`/`other` never touch the pantry. A picked **plan-derived** line has **no row to remove** — don't hunt for one; its pantry restock is what clears it from the next derivation. Then offer a couple of storage tips for fresh perishables just received, following the **Putting groceries away** guidance.
