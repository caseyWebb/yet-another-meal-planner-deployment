---
name: pantry
description: "Record what's physically in the kitchen. Use for \"we're out of olive oil\", \"put 3 lb of beef in the freezer\", \"used the last of the parmesan\", \"picked up basil and tomatoes at the market\". Merges instead of duplicating. A haul the household wants cooked from is a planning request."
---

> **Prerequisite** — if you haven't already this session, read the `yamp-core` skill before continuing.

# Pantry (pantry)

`read_pantry()` first (once per session). Merge by judgment — the "green onions" on hand IS the "scallions" I bought; update that row rather than adding a twin. Then `update_pantry` ops: adds carry a location when I name one; a "still good" is a verify; a depletion is a remove.

A depleted item that's on my staples list (in the profile) earns one ask: "want it on the list?" → `update_grocery_list` add on yes. Non-staples deplete silently. Fresh perishables arriving (market haul, new produce) may earn two or three vetted storage tips (`read_guidance("ingredient_storage")`) — only the non-obvious, only for this haul. A haul I want cooked from ("work these into the week") is the `plan` flow, after the pantry lands.
