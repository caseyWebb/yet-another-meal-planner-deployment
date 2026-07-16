---
name: cook
description: "Walk the household through cooking hands-free, or capture a meal already made. Use for \"I'm making the arroz caldo\", \"walk me through dinner\", \"let's cook\" — and equally for \"I made the chili last night\", \"we ate the leftovers\". One flow owns the cooking log."
---

> **Prerequisite** — if you haven't already this session, read the `yamp-core` skill before continuing.

# Cook (cook)

Identify the dish: check `read_meal_plan()` first (a loose match to a planned dish counts — the plan has a soup and I say "made the soup"); otherwise a vibe-less `search_recipes` query lookup, or treat it as an off-cookbook meal.

**Cooking now:** pre-flight in chat, short turns — equipment the dish truly needs (against the profile's kitchen; gear I volunteer gets saved silently via `update_pantry` kitchen ops), the gather list, pin servings, and whether anything's short. Then `display_recipe` — its cook mode carries the steps, check-offs, and timers (I run timers, you never do); no card on this surface → a plain-text walk, one step per turn. Weave in a saved technique (`read_guidance("cooking_techniques")`) at the step where it applies, if one genuinely fits.

**Capture — during the walk's finish, or a past-tense report:** `log_cooked` — `{ type: "recipe", recipe, plan_row_id? }` for a cookbook dish (its plan row clears itself), `{ type: "ad_hoc", name, protein?, cuisine? }` otherwise; honest `meal` when known, past `date` when I said so. Ask what ran out ("did that finish the ginger?") → `update_pantry` removes. Then one light ask: favorite it, or a note for next time? (`set_recipe_disposition` / `add_recipe_note`.) Don't propose a new menu unless I ask.
